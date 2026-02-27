---
title: 20210628 - DDL 过程梳理
confluence_page_id: 1147046
created_at: 2021-06-28T05:17:28+00:00
updated_at: 2021-06-28T05:17:28+00:00
---

版本: MySQL 8.0

入口: mysql_alter_table

# 代码流程

## mysql_prepare_alter_table

## 创建临时表结构

  - 不记录到binlog

  - create_table_impl

## 对于 inplace/instant 类型

  1. 准备alter的元信息: fill_alter_inplace_info

  2. 打开之前创建的临时表, 根据alter的元信息 设置参数

  3. 询问引擎的支持状况: inplace_supported = check_if_supported_inplace_alter(...)

     - 支持inplace的三大类: INNOBASE_INPLACE_IGNORE, INNOBASE_ALTER_NOREBUILD, INNOBASE_ALTER_REBUILD

     - 对于三大类之外的情况, 以及三大类中的特殊子情况进行判断, 对于不支持inplace的返回HA_ALTER_INPLACE_NOT_SUPPORTED, 使用策略

     - 判断是否支持instant: innobase_support_instant

       - 按操作类型, 返回支持度

         - INSTANT_IMPOSSIBLE

         - INSTANT_ADD_COLUMN

           - ADD_STORED_BASE_COLUMN

           - ADD_VIRTUAL_COLUMN

         - INSTANT_NO_CHANGE

         - INSTANT_VIRTUAL_ONLY

       - support_instant_add: 非压缩表/非系统表/非临时表 可支持 INSTANT_ADD_COLUMN

     - 判断是否适用于online

       - online的情况: 返回 HA_ALTER_INPLACE_NO_LOCK_AFTER_PREPARE

       - 非online的情况: 返回 HA_ALTER_INPLACE_SHARED_LOCK_AFTER_PREPARE

  4. 将引擎的支持状况, 与SQL中的指定的LOCK, 进行兼容性检查

  5. mysql_inplace_alter_table

     1. prepare 阶段按需上表锁 (对旧表)
            
            Upgrade to EXCLUSIVE lock if:
            - This is requested by the storage engine
            - Or the storage engine needs exclusive lock for just the prepare
              phase
            - Or requested by the user
            
            Upgrade to SHARED_NO_WRITE lock if:
            - The storage engine needs writes blocked for the whole duration
            - Or this is requested by the user

     2. online = HA_ALTER_INPLACE_NO_LOCK | HA_ALTER_INPLACE_NO_LOCK_AFTER_PREPARE

     3. ha_prepare_inplace_alter_table

        1. 复制auto increment结构

        2. prepare_inplace_alter_table_impl

           1. 对于不需要修改数据(no ALTER_DATA)的情况:   
(INNOBASE_ALTER_DATA = INNOBASE_ONLINE_CREATE | INNOBASE_ALTER_REBUILD)

              1. dd_prepare_inplace_alter_table

                 1. 为table_per_file的情况, 删除旧的表空间元数据, 建立新的表空间元数据

           2. 对于需要修改数据的情况

              - prepare_inplace_alter_table_dict

                - 开启事务: trx_start_if_not_started_xa

                - 对于非online的情况, 对新表上表锁: row_merge_lock_table

                - 对于重建聚簇索引, 将使用临时表进行重建, 此处创建临时表结构: If a new clustered index is defined for the table we need to rebuild the table with a temporary name

                  - dict_mem_table_create

                  - row_create_table_for_mysql

                  - dd_table_open_on_name_in_mem

                  - 处理ADD_COLUMN

                  - innobase_build_col_map

                - 处理索引增加: Create the indexes and load into dictionary

                - Allocate a log for online table rebuild: 设置 index->online_log

                - 对于online的情况, trx_assign_read_view

                - dd_prepare_inplace_alter_table

     4. prepare阶段结束, 按需降锁

     5. ha_inplace_alter_table

        - instant, 不需要此阶段

        - 不涉及INNOBASE_ALTER_DATA, 不需要此阶段

        - row_merge_build_indexes: Read the clustered index of the table and build indexes based on this information using temporary files and merge sort

          - 大致流程:

            - 对于新表的每一个索引:

              - 按照聚簇索引, 读取数据块

              - 将数据块按照新索引进行排序 (如果新索引是聚簇索引, 跳过此步骤)

              - 将 排序好的数据块 与 新索引的数据 进行合并排序

        - 对于online回放: row_log_table_apply

          - row_log_table_apply_ops

            - 从临时文件中不断读取block, 交给row_log_table_apply_op处理

              - block格式为:
                    
                    1 (op type) + 1 (extra_size) + at least 1 byte payload

              - 分为INSERT/UPDATE/DELETE分别处理

     6. wait_while_table_is_used: Upgrade to EXCLUSIVE before commit.

     7. ha_commit_inplace_alter_table: 从内存结构dict_table_t 写入 DD结构??

        1. commit_inplace_alter_table_impl: 对于非instant/非INNOBASE_INPLACE_IGNORE, 需要更新表元信息:

           1. rollback操作: rollback_inplace_alter_table

           2. 对于INNOBASE_INPLACE_IGNORE或instant: 不作处理

           3. Generate the temporary name for old table, and acquire mdl  
lock on it

           4. ** *Latch the InnoDB data dictionary exclusively*** so that no deadlocks  
or lock waits can happen in it during the data dictionary operation

           5. Prevent the background statistics collection from accessing  
the tables

           6. Apply the changes to the data dictionary tables, for all  
partitions

              1. rebuild: commit_try_rebuild

                 1. 提交前检查

                 2. 对于online, 回放最后一点DDL log: row_log_table_apply (此时表锁已升级到EXCLUSIVE)

              2. no-rebuild: commit_try_norebuild

                 1. 提交前检查

           7. Commit or roll back the changes to the data dictionary

              1. rebuild: commit_cache_rebuild  
Apply the changes made during commit_try_rebuild(),  
to the data dictionary cache and the file system

                 1. InnoDB旧表 更名为 临时名字

                 2. InnoDB新表 更名为 InnoDB旧表

              2. no-rebuild: commit_cache_norebuild:   
Commit the changes to the data dictionary cache  
after a successful commit_try_norebuild() call

                 1. 将删除的列/索引, 从DD cache中移除

        2. 对于instant: dd_commit_inplace_instant

           1. INSTANT_NO_CHANGE: dd_commit_inplace_no_change

              1. dd_copy_private: 复制引擎私有数据

              2. dd_copy_table

                 1. 复制表列信息

                 2. 复制INSTANT ADD COLUMN信息

           2. INSTANT_VIRTUAL_ONLY:

              1. dd_commit_inplace_no_change

              2. 对virtual列的处理

              3. innobase_discard_table

           3. INSTANT_ADD_COLUMN

              1. dd_copy_private

              2. dd_commit_instant_table

              3. innobase_discard_table

        3. 否则: 对于INNOBASE_INPLACE_IGNORE: dd_commit_inplace_no_change

        4. 否则: dd_commit_inplace_alter_table TODO

           1. 对于非rebuild, 还需要调用: dd_commit_inplace_update_instant_meta

              1. 写instant cols

              2. 写default value

     8. Replace table definition in the data-dictionary

     9. Rename altered table if requested

     10. write_bin_log

     11. commit

     12. reopen table

        1. reload InnoDB-level DD from server-level DD

     13. Finally we can tell SE supporting atomic DDL that the changed table in the data-dictionary

## 对于copy类型

  - 引擎创建临时表: ha_create_table

  - copy_data_between_tables

  - Data is copied. Now we:
        
        1) Wait until all other threads will stop using old version of table
               by upgrading shared metadata lock to exclusive one.
        2) Close instances of table open by this thread and replace them
               with placeholders to simplify reopen process.
        3) Rename the old table to a temp name, rename the new one to the
               old name.
        4) If we are under LOCK TABLES and don't do ALTER TABLE ... RENAME
               we reopen new version of table.
        5) Write statement to the binary log.
        6) If we are under LOCK TABLES and do ALTER TABLE ... RENAME we
               remove placeholders and release metadata locks.
        7) If we are not not under LOCK TABLES we rely on the caller
              (mysql_execute_command()) to release metadata locks.

## write_bin_log

## commit

# 不在代码分析中讨论的维度

  1. add/remove functional index

  2. trigger/view

  3. foreign key

  4. FTS/GIS

  5. 非InnoDB引擎, 包括MERGE引擎和分区表

  6. 修改tablespace, file per table

  7. LOCK TABLE 与 ALTER 的联动

  8. 升级兼容性

  9. 条件检查

  10. 虚拟列

  11. CHANGE_CREATE_OPTION

  12. 对histograms的处理

# 需要分析的五个维度

Instant  
| (切换表定义?) | In Place | Rebuilds Table | Permits Concurrent DML (online) | Only Modifies Metadata | Example |
| --- | --- | --- | --- | --- | --- |
| Yes | Yes | Yes | Yes | Yes |  |
| Yes | Yes | Yes | Yes | No |  |
| Yes | Yes | Yes | No | Yes |  |
| Yes | Yes | Yes | No | No |  |
| Yes | Yes | No | Yes | Yes | Setting a column default value |
| Yes | Yes | No | Yes | No |  |
| Yes | Yes | No | No | Yes |  |
| Yes | Yes | No | No | No |  |
| Yes | No | Yes | Yes | Yes | - |
| Yes | No | Yes | Yes | No | - |
| Yes | No | Yes | No | Yes | - |
| Yes | No | Yes | No | No | - |
| Yes | No | No | Yes | Yes | - |
| Yes | No | No | Yes | No | - |
| Yes | No | No | No | Yes | - |
| Yes | No | No | No | No | - |
| No | Yes | Yes | Yes | Yes |  |
| No | Yes | Yes | Yes | No | Dropping a column |
| No | Yes | Yes | No | Yes |  |
| No | Yes | Yes | No (???) | No | Specifying a character set |
| No | Yes | No | Yes | Yes | Extending VARCHAR column sizeTODO No Instant?//Dropping an index |
| No | Yes | No | Yes | No | Creating or adding a secondary index |
| No | Yes | No | No | Yes |  |
| No | Yes | No | No | No | 不存在的情况 |
| No | No | Yes | Yes | Yes |  |
| No | No | Yes | Yes | No | 不存在的情况:COPY策略不允许并发DML |
| No | No | Yes | No | Yes |  |
| No | No | Yes | No | No | Changing the column data type |
| No | No | No | Yes | Yes |  |
| No | No | No | Yes | No |  |
| No | No | No | No | Yes |  |
| No | No | No | No | No |  |
  
结论:

  1. 当Instant = Yes时, Only Modifies Metadata = Yes.   
例外: “Adding a column”(有条件的Instant, 不满足Instant时, 需要ALTER_DATA)

  2. 当Instant = Yes时, Permits Concurrent DML = Yes

  3. 当Rebuilds Table = Yes时, Only Modifies Metadata = No

  4. 当Only Modifies Metadata = Yes, Permits Concurrent DML = Yes

  5. “所谓Inplace，也就是索引创建在原表上直接进行，不会拷贝临时表。相对于Copy Table方式，这是一个进步”

# 比较INPLACE: INSTANT与非INSTANT的代码路径差异

| INSTANT | In-place, no INSTANT |  |
| --- | --- | --- | prepare前上锁
| mdl_context.upgrade_shared_lock| 不上锁| 上锁
| prepareprepare_inplace_alter_table_imp| 无prepare| prepare
| prepare后锁降级mdl_ticket->downgrade_lock| 不上锁| 上锁
| alterinplace_alter_table_impl| 无alter| alter
| 锁升级| EXCLUDE| EXCLUDE
| commitcommit_inplace_alter_table_impl| 无commit| commit
| dd_commit_*| dd_commit_inplace_instant| dd_commit_inplace_no_changedd_commit_inplace_alter_table
| log binlogha_binlog_log_query| 有| 有
| tx committrans_commit_stmt| 有| 有
  
# 读代码的步骤

## ALTER_COLUMN_DEFAULT

  1. handler.h 找到定义: ALTER_COLUMN_DEFAULT

  2. [handler0alter.cc](<http://handler0alter.cc>) 找到InnoDB策略: ALTER_COLUMN_DEFAULT 属于 INNOBASE_INPLACE_IGNORE

  3. [handler0alter.cc](<http://handler0alter.cc>) 找到 inplace策略: handler::check_if_supported_inplace_alter, 检查是否支持inplace

     1. [handler0alter.cc](<http://handler0alter.cc>) 找到instant策略: innobase_support_instant. INNOBASE_INPLACE_IGNORE 对应 INSTANT_NO_CHANGE

     2. INSTANT_NO_CHANGE 对应 HA_ALTER_INPLACE_INSTANT

## DROP_STORED_COLUMN

  1. handler.h 找到定义: DROP_STORED_COLUMN

  2. [handler0alter.cc](<http://handler0alter.cc>) 找到InnoDB策略: DROP_STORED_COLUMN 属于 INNOBASE_ALTER_REBUILD

  3. [handler0alter.cc](<http://handler0alter.cc>) 找到 inplace策略: handler::check_if_supported_inplace_alter, 检查是否支持inplace. 返回: HA_ALTER_INPLACE_NO_LOCK_AFTER_PREPARE

## Extending VARCHAR column size

  1. ha_alter_info->handler_flags 为 ALTER_COLUMN_EQUAL_PACK_LENGTH | ALTER_COLUMN_DEFAULT

  2. innobase_rename_or_enlarge_columns_cache: 更新: data dictionary cache

## DROP_INDEX

  1. commit_cache_norebuild

     1. 更新 data dictionary cache

     2. 更新 Adaptive Hash Index

# 遗留问题

  1. 引擎如何进行原子性的DDL TODO

  2. “Setting a column default value” 例子更换为 rename table ??TODO

  3. 分配online DDL log后, 如何保障rebuild与DML互不干扰  
rebuild过程通过MVCC R-R读取数据, 使得与并发的DML互不干扰:
         
         /* Perform a REPEATABLE READ.
         
               When rebuilding the table online,
               row_log_table_apply() must not see a newer
               state of the table when applying the log.
               This is mainly to prevent false duplicate key
               errors, because the log will identify records
               by the PRIMARY KEY, and also to prevent unsafe
               BLOB access.
         
               When creating a secondary index online, this
               table scan must not see records that have only
               been inserted to the clustered index, but have
               not been written to the online_log of
               index[]. If we performed READ UNCOMMITTED, it
               could happen that the ADD INDEX reaches
               ONLINE_INDEX_COMPLETE state between the time
               the DML thread has updated the clustered index
               but has not yet accessed secondary index. */

  4. inplace rebuild和copy的策略比较 (drop column):

     1. 成本比较:

        1. 维护RR read view的redo log成本

        2. DDL log的成本 (格式?)

        3. rebuild/copy 形成新表的redo log成本

        4. 新表存储成本
               
               When an operation on the primary key uses ALGORITHM=INPLACE, even though the data is still copied, it is more efficient than using ALGORITHM=COPY because:
               No undo logging or associated redo logging is required for ALGORITHM=INPLACE. These operations add overhead to DDL statements that use ALGORITHM=COPY.
               The secondary index entries are pre-sorted, and so can be loaded in order.
               The change buffer is not used, because there are no random-access inserts into the secondary indexes.

  5. “Specifying a character set”:

     1. 调试结果为online, 但标注为非online

     2. 为什么需要重建表?

  6. inplace no rebuild 与 instant 的策略比较 (Extending VARCHAR column size)

     1. 参考: <https://dev.mysql.com/worklog/task/?id=6554>

     2. 猜想: instant指的是不需要变更InnoDB层data dictionary中的元数据, 只变更SQL层的dd中的元数据  
inplace+no rebuild是需要变更InnoDB层的data dictionary中的元数据

        1. SQL层的DD结构: mysql-8.0.15/sql/dd/impl/types/column_impl.h

        2. InnoDB层的DD结构: mysql-8.0.15/storage/innobase/include/dict0mem.h

     3. 问题1: 以Setting a column default value为例, DDL过程更新了server层的DD, 那么InnoDB层的DD何时知道此变更

        1. commit后的reopen table

     4. 问题2: 对于Extending VARCHAR column size, 有什么信息是必须要直接提交到InnoDB层的DD, 而不能用reopen table从server的DD中读取到InnoDB的DD中

        1. col->len / col->prtype / col->mbminmaxlen ??

# 参考

  1. <http://www.cnblogs.com/cchust/p/4639397.html>

  2. <http://hedengcheng.com/?p=405> TODO

  3. <https://dev.mysql.com/worklog/task/?id=11250>

  4. server层DD与InnoDB dict_*的关系: <https://mysqlserverteam.com/the-unified-data-dictionary-lab-release/>

     1. server->InnoDB 的元数据映射: dd_fill_dict_table, fill_columns_from_dd  
dd_fill_dict_table 调用堆栈:
            
            #0  dd_open_table_one (client=0x7f8b840047a0, table=0x7f8b84485c10, norm_name=0x7f8bc0097250 "test/test", dd_table=0x7f8b84158678, thd=0x7f8b84001150, fk_list=std::deque with 0 elements) at /opt/mysql-8.0.15/storage/innobase/dict/dict0dd.cc:3948
            #1  0x000055fec62090f6 in dd_open_table  (client=0x7f8b840047a0, table=0x7f8b84485c10, norm_name=0x7f8bc0097250 "test/test", dd_table=0x7f8b84158678, thd=0x7f8b84001150) at /opt/mysql-8.0.15/storage/innobase/dict/dict0dd.cc:4270
            #2  0x000055fec5daa922 in ha_innobase::open (this=0x7f8b84486638, name=0x7f8b84163a10 "./test/test", open_flags=2, table_def=0x7f8b84158678) at /opt/mysql-8.0.15/storage/innobase/handler/ha_innodb.cc:6567
            #3  0x000055fec4a9724b in handler::ha_open (this=0x7f8b84486638, table_arg=0x7f8b84485c10, name=0x7f8b84163a10 "./test/test", mode=2, test_if_locked=2, table_def=0x7f8b84158678) at /opt/mysql-8.0.15/sql/handler.cc:2650
            #4  0x000055fec489ad93 in open_table_from_share (thd=0x7f8b84001150, share=0x7f8b841636b8, alias=0x7f8b840483b8 "test", db_stat=39, prgflag=8, ha_open_flags=0, outparam=0x7f8b84485c10, is_create_table=false, table_def_param=0x7f8b84158678) at /opt/mysql-8.0.15/sql/table.cc:3070
            #5  0x000055fec4696e8c in open_table (thd=0x7f8b84001150, table_list=0x7f8bc0097ba0, ot_ctx=0x7f8bc0097b40) at /opt/mysql-8.0.15/sql/sql_base.cc:3345
            #6  0x000055fec480b77d in mysql_inplace_alter_table (thd=0x7f8b84001150, schema=..., new_schema=..., table_def=0x0, altered_table_def=0x7f8b84185a98, table_list=0x7f8b84048800, table=0x0, altered_table=0x7f8b840e2400, ha_alter_info=0x7f8bc0098460, inplace_supported=HA_ALTER_INPLACE_INSTANT, alter_ctx=0x7f8bc0098e50, columns=std::set with 1 element = {...}, fk_key_info=0x7f8b841aaf18, fk_key_count=0, fk_invalidator=0x7f8bc00983a0) at /opt/mysql-8.0.15/sql/sql_table.cc:12302
            #7  0x000055fec48157cc in mysql_alter_table (thd=0x7f8b84001150, new_db=0x7f8b84048da0 "test", new_name=0x0, create_info=0x7f8bc009a180, table_list=0x7f8b84048800, alter_info=0x7f8bc009a280) at /opt/mysql-8.0.15/sql/sql_table.cc:15476
            #8  0x000055fec4d01707 in Sql_cmd_alter_table::execute (this=0x7f8b84048e78, thd=0x7f8b84001150) at /opt/mysql-8.0.15/sql/sql_alter.cc:343
            #9  0x000055fec4750a7c in mysql_execute_command (thd=0x7f8b84001150, first_level=true) at /opt/mysql-8.0.15/sql/sql_parse.cc:4354
            #10 0x000055fec47531d1 in mysql_parse (thd=0x7f8b84001150, parser_state=0x7f8bc009bc80, force_primary_storage_engine=false) at /opt/mysql-8.0.15/sql/sql_parse.cc:5105
            #11 0x000055fec47489e3 in dispatch_command (thd=0x7f8b84001150, com_data=0x7f8bc009cc90, command=COM_QUERY) at /opt/mysql-8.0.15/sql/sql_parse.cc:1715
            #12 0x000055fec4746f39 in do_command (thd=0x7f8b84001150) at /opt/mysql-8.0.15/sql/sql_parse.cc:1263
            #13 0x000055fec48fbe63 in handle_connection (arg=0x55fec93ad3e0) at /opt/mysql-8.0.15/sql/conn_handler/connection_handler_per_thread.cc:305
            #14 0x000055fec63d042e in pfs_spawn_thread (arg=0x55fec946b630) at /opt/mysql-8.0.15/storage/perfschema/pfs.cc:2836
            #15 0x00007f8bd7cc26db in start_thread (arg=0x7f8bc009d700) at pthread_create.c:463
            #16 0x00007f8bd63bc88f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

  5. MySQL 注释:
         
         /* On-line/in-place/instant ALTER TABLE interface. */
         
           /*
             Here is an outline of on-line/in-place ALTER TABLE execution through
             this interface.
         
             Phase 1 : Initialization
             ========================
             During this phase we determine which algorithm should be used
             for execution of ALTER TABLE and what level concurrency it will
             require.
         
             *) This phase starts by opening the table and preparing description
                of the new version of the table.
             *) Then we check if it is impossible even in theory to carry out
                this ALTER TABLE using the in-place/instant algorithm. For example,
                because we need to change storage engine or the user has explicitly
                requested usage of the "copy" algorithm.
             *) If in-place/instant ALTER TABLE is theoretically possible, we continue
                by compiling differences between old and new versions of the table
                in the form of HA_ALTER_FLAGS bitmap. We also build a few
                auxiliary structures describing requested changes and store
                all these data in the Alter_inplace_info object.
             *) Then the handler::check_if_supported_inplace_alter() method is called
                in order to find if the storage engine can carry out changes requested
                by this ALTER TABLE using the in-place or instant algorithm.
                To determine this, the engine can rely on data in HA_ALTER_FLAGS/
                Alter_inplace_info passed to it as well as on its own checks.
                If the in-place algorithm can be used for this ALTER TABLE, the level
                of required concurrency for its execution is also returned.
                If any errors occur during the handler call, ALTER TABLE is aborted
                and no further handler functions are called.
                Note that in cases when there is difference between in-place and
                instant algorithm and user explicitly asked for usage of in-place
                algorithm storage engine MUST return one of values corresponding
                to in-place algorithm and not HA_ALTER_INPLACE_INSTANT from this
                method.
             *) Locking requirements of the in-place algorithm are compared to any
                concurrency requirements specified by user. If there is a conflict
                between them, we either switch to the copy algorithm or emit an error.
         
             Phase 2 : Execution
             ===================
         
             In this phase the operations are executed.
         
             *) As the first step, we acquire a lock corresponding to the concurrency
                level which was returned by handler::check_if_supported_inplace_alter()
                and requested by the user. This lock is held for most of the
                duration of in-place ALTER (if HA_ALTER_INPLACE_SHARED_LOCK_AFTER_PREPARE
                or HA_ALTER_INPLACE_NO_LOCK_AFTER_PREPARE were returned we acquire an
                exclusive lock for duration of the next step only).
                For HA_ALTER_INPLACE_INSTANT we keep shared upgradable metadata lock
                which was acquired at table open time.
             *) After that we call handler::ha_prepare_inplace_alter_table() to give the
                storage engine a chance to update its internal structures with a higher
                lock level than the one that will be used for the main step of algorithm.
                After that we downgrade the lock if it is necessary.
                This step should be no-op for instant algorithm.
             *) After that, the main step of this phase and algorithm is executed.
                We call the handler::ha_inplace_alter_table() method, which carries out
                the changes requested by ALTER TABLE but does not makes them visible to
                other connections yet.
                This step should be no-op for instant algorithm as well.
             *) We ensure that no other connection uses the table by upgrading our
                lock on it to exclusive.
             *) a) If the previous step succeeds,
             handler::ha_commit_inplace_alter_table() is called to allow the storage
             engine to do any final updates to its structures, to make all earlier
             changes durable and visible to other connections.
             For instant algorithm this is the step during which SE changes are done.
             Engines that support atomic DDL only prepare for the commit during this
             step but do not finalize it. Real commit happens later when the whole
             statement is committed. Also in some situations statement might be rolled
             back after call to commit_inplace_alter_table() for such storage engines.
             In the latter special case SE might require call to
             handlerton::dict_cache_reset() in order to invalidate its internal table
             definition cache after rollback.
             b) If we have failed to upgrade lock or any errors have occured during
             the handler functions calls (including commit), we call
             handler::ha_commit_inplace_alter_table() to rollback all changes which
             were done during previous steps.
         
             All the above calls to SE are provided with dd::Table objects describing old
             and new version of table being altered. Engines which support atomic DDL are
             allowed to adjust object corresponding to the new version. During phase 3
             these changes are saved to the data-dictionary.
         
         
             Phase 3 : Final
             ===============
         
             In this phase we:
         
             a) For engines which don't support atomic DDL:
         
                *) Update the SQL-layer data-dictionary by replacing description of old
                   version of the table with its new version. This change is immediately
                   committed.
                *) Inform the storage engine about this change by calling the
                   handler::ha_notify_table_changed() method.
                *) Process the RENAME clause by calling handler::ha_rename_table() and
                   updating the data-dictionary accordingly. Again this change is
                   immediately committed.
                *) Destroy the Alter_inplace_info and handler_ctx objects.
         
             b) For engines which support atomic DDL:
         
                *) Update the SQL-layer data-dictionary by replacing description of old
                   version of the table with its new version.
                *) Process the RENAME clause by calling handler::ha_rename_table() and
                   updating the data-dictionary accordingly.
                *) Commit the statement/transaction.
                *) Finalize atomic DDL operation by calling handlerton::post_ddl() hook
                   for the storage engine.
                *) Additionally inform the storage engine about completion of ALTER TABLE
                   for the table by calling the handler::ha_notify_table_changed()
             method.
                *) Destroy the Alter_inplace_info and handler_ctx objects.
           */
