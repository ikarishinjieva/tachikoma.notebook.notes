---
title: 20220422 - 国债故障 2
confluence_page_id: 1737101
created_at: 2022-04-22T15:48:56+00:00
updated_at: 2022-04-28T07:17:33+00:00
---

# 目标

搞清楚为什么 SYS_TABLES 的 数据页, 会在 buffer pool里设置成 锁定页, 并且IO类型为NONE

![image2022-4-22 23:26:26.png](/assets/01KJBYJCDGQG0YRF8688HMW9VH/image2022-4-22%2023%3A26%3A26.png)

![image2022-4-23 23:54:18.png](/assets/01KJBYJCDGQG0YRF8688HMW9VH/image2022-4-23%2023%3A54%3A18.png)

# 现象

  1. SYS_TABLES 刷盘会刷到 innodb temporary table space, temporary table space 特别大 (27G), 但大部分都是空闲空间 (曾经有27G的需求)
  2. redo log 正常向前推进刷盘, LSN number并不受 SYS_TABLES 滞留flush_list的影响
  3. 国债出问题的是slave, 但运行了很久, 以前是master. 
  4. 在国债当前的master上, 也发现了SYS_TABLES滞留, 数量随着运行时间上涨, 也就是说, 可能是缓慢泄露
  5. page number是连续的

# 代码分析: 找到fix_count 加减不对称的函数

  - MySQL版本 5.7.29
  - 入手, 从buf_page_t -> buf_fix_count的变更函数开始
    - +1
      - buf_page_init: os_atomic_increment_uint32
      - buf_page_init_for_read: os_atomic_increment_uint32
      - buf_block_fix
        - buf_pool_watch_set: 
          - 函数返回的buf_page都会 fix_count + 1
          - 调用方: 
            - buf_page_get_gen: 放在后面分析
        - buf_page_get_zip
          - 暂不调查
        - buf_page_get_gen  

          - 在 buf0buf.cc:4161 处, 调用了buf_pool_watch_set后, 又进行了 buf_block_fix, 导致了两次fix
            - 与 BUF_GET_IF_IN_POOL_OR_WATCH 有关, 暂不调查
          - 在 [buf0buf.cc](<http://buf0buf.cc>):4243 处, 调用了 buf_block_fix, 对找到的页进行 fix_count+1
          - 函数返回的buf_page, 都会 fix_count+1
          - 调用方: ???
        - buf_block_buf_fix_inc_func = buf_block_buf_fix_inc
          - PageBulk::release
            - bulk load, 暂不调查
          - btr_cur_optimistic_latch_leaves
            - 函数返回的buf_page, 都会 fix_count+1
            - 调用方: 
              - btr_pcur_restore_position_func: 有一部分分支不确定 fix_count的效果 (部分子函数未分析)
          - btr_blob_log_check_t.check
            - 与BTR_STORE_INSERT_BULK逻辑先关, 函数内封闭, 没有对外影响. 但有疑虑
          - buf_page_get_zip
            - 暂不调查
          - buf_page_optimistic_get
            - 函数返回的buf_page, 都会 fix_count+1
            - 调用方: 
              - btr_cur_optimistic_latch_leaves: 已查
          - buf_page_get_known_nowait
            - 函数返回的buf_page, 都会 fix_count+1
            - 调用方: 
              - btr_search_guess_on_hash
              - ibuf_merge_or_delete_for_page * 2
          - buf_page_try_get_func = buf_page_try_get
            - 函数返回的buf_page, 都会 fix_count+1
            - 调用方: 
              - fill_lock_data
              - lock_rec_print
          - buf_page_create
            - 函数返回的buf_page, 都会 fix_count+1
            - 调用方: ???
          - fsp_page_create
            - buf_page_create 返回的block, fix_count已经+1
            - 而 fsp_page_create 还会再次+1
            - 通过调试, 已确认返回的block->page->fix_count=2
            - 但对于一个block, 该函数只调用一次, 不会让fix_count过大, 与现象不符, 暂不调查
    - -1
      - buf_block_unfix
        - memo_slot_release
          - mtr_t::memo_release
            - btr_leaf_page_release
          - ...
    - =0
      - ...

规模比较庞大, 不确定能走通

# 从SYS_TABLES的变更触发

  - SYS_TABLES
    - dict_boot
      - 创建了SYS_TABLES, 以及两个相关索引
    - dict_create_sys_tables_tuple
      - 创建 SYS_TABLES 的行
      - dict_build_table_def_step
        - dict_create_table_step
          - que_thr_step
    - dict_space_is_empty
      - 检查 space_id是否在SYS_TABLES中存在, 使用 dict_startscan_system 扫描 SYS_TABLES
      - 调用方: innobase_drop_tablespace
      - mtr封闭
    - dict_get_first_table_name_in_db
      - 无mtr
      - row_drop_database_for_mysql
      - row_rename_partitions_for_mysql
    - dict_load_table_low  

      - 无mtr
      - dict_load_table_one
        - dict_load_table
          - dict_table_open_on_name
          - dict_load_table_on_id
          - dict_table_get_low
          - innobase_update_foreign_cache
          - row_table_add_foreign_constraints
          - row_drop_table_from_cache
          - row_mysql_drop_temp_tables
          - row_rename_table_for_mysql
    - 归类操作: 
      - rename table
        - row_rename_table_for_mysql
          - 带有事务, 不封闭, 可调查 调用方
        - 常规表, rename 临时表会报错
      - drop table
        - row_drop_table_for_mysql
          - 带有事务, 不封闭, 可调查 调用方
        - 常规表 / 临时表
      - alter table ... ALGORITHM=INPLACE
        - row_merge_rename_tables_dict
          - 带有事务, 不封闭, 可调查 调用方
      - i_s_sys_tables_fill_table: discard/import tablespace
        - mtr 封闭
        - row_import_update_discarded_flag
          - 带有事务, 不封闭, 可调查 调用方
        - row_mysql_table_id_reassign
          - 带有事务, 不封闭, 可调查 调用方
        - row_import_for_mysql / dict_get_and_save_data_dir_path
      - truncate table
        - row_truncate_update_table_id
          - 带有事务, 不封闭, 可调查 调用方
          - row_truncate_update_system_tables
      - InnoDB 启动: 
        - row_truncate_update_table_id
          - 带有事务, 不封闭, 可调查 调用方
          - row_truncate_update_sys_tables_during_fix_up
            - fixup_tables_in_system_tablespace/fixup_tables_in_non_system_tablespace
              - innobase_start_or_create_for_mysql
        - dict_sys_tables_rec_read / dict_sys_tables_rec_check  

          - dict_check_sys_tables
            - dict_check_tablespaces_and_store_max_id
              - innobase_start_or_create_for_mysql
      - crash recovery
        - row_mysql_drop_temp_tables
          - mtr封闭
        - fts_update_hex_format_flag
          - 带有事务, 不封闭, 可调查 调用方
          - fts_drop_orphaned_tables
            - recv_recovery_rollback_active
      - information_schema  

        - dict_sys_tables_rec_read / dict_sys_tables_rec_check  

          - dict_load_table_low
          -             - dict_process_sys_tables_rec_and_mtr_commit  

              - i_s_sys_tables_fill_table: select * from information_schema.innodb_sys_tables
              - i_s_sys_tables_fill_table_stats: select * from information_schema.innodb_sys_tables
      - alter table ... add virtual column
        - innobase_update_n_virtual
          - 带有事务, 不封闭, 可调查 调用方

  - 检查 dict_startscan_system 的调用, 是否mtr封闭: 全部封闭

# 逐个阅读dict0*文件

看看有没有函数逻辑异常, 能多次读取SYS_TABLES中的数据页并锁定

线索: 

  - dict_disable_redo_if_temporary
  - dict_startscan_system
  - dict_getnext_system
  - dict_table_check_if_in_cache_low
  - SYS_TABLES 读取过程在 dict0load 中, 写入过程 参看: tab_create_graph_create

# 检查正常INSERT数据时, 为什么buffer pool中的 SYS_TABLES 页会增加

断点: buf_page_init_low, 可获取bpage的地址

```
(gdb) bt
#0  buf_page_init_low (bpage=0x7f81b6f50300)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0buf.cc:5001
#1  0x0000000001e90a27 in buf_page_init (buf_pool=0x5504e38, page_id=...,
    page_size=..., block=0x7f81b6f50300)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0buf.cc:5053
#2  0x0000000001e91cf5 in buf_page_create (page_id=..., page_size=...,
    mtr=0x7f816d08e520)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0buf.cc:5407
#3  0x0000000001f28fc2 in fsp_page_create (page_id=..., page_size=...,
    rw_latch=RW_X_LATCH, mtr=0x7f816d08e520, init_mtr=0x7f816d08e520)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/fsp/fsp0fsp.cc:1871
#4  0x0000000001f2dab9 in fseg_alloc_free_page_low (space=0x5bc6768,
    page_size=..., seg_inode=0x7f81b78ed8f2 "", hint=4357, direction=111 'o',
    rw_latch=RW_X_LATCH, mtr=0x7f816d08e520, init_mtr=0x7f816d08e520,
    has_done_reservation=1)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/fsp/fsp0fsp.cc:3195
#5  0x0000000001f2dc45 in fseg_alloc_free_page_general (
    seg_header=0x7f81b83a804a "", hint=4357, direction=111 'o',
    has_done_reservation=1, mtr=0x7f816d08e520, init_mtr=0x7f816d08e520)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/fsp/fsp0fsp.cc:3263
#6  0x0000000001e3ed0f in btr_page_alloc_low (index=0x7f816cf06720,
    hint_page_no=4357, file_direction=111 'o', level=0, mtr=0x7f816d08e520,
    init_mtr=0x7f816d08e520)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/btr/btr0btr.cc:469
#7  0x0000000001e3ed8c in btr_page_alloc (index=0x7f816cf06720,
    hint_page_no=4357, file_direction=111 'o', level=0, mtr=0x7f816d08e520,
    init_mtr=0x7f816d08e520)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/btr/btr0btr.cc:502
#8  0x0000000001e441c9 in btr_page_split_and_insert (flags=3,
--Type <RET> for more, q to quit, c to continue without paging--
    cursor=0x7f81ac4498b0, offsets=0x7f81ac449980, heap=0x7f81ac449988,
    tuple=0x7f816ceed370, n_ext=0, mtr=0x7f816d08e520)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/btr/btr0btr.cc:2639
#9  0x0000000001e5c4a3 in btr_cur_pessimistic_insert (flags=3,
    cursor=0x7f81ac4498b0, offsets=0x7f81ac449980, heap=0x7f81ac449988,
    entry=0x7f816ceed370, rec=0x7f81ac449978, big_rec=0x7f81ac449970,
    n_ext=0, thr=0x7f816cba1f28, mtr=0x7f816d08e520)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/btr/btr0cur.cc:3458
#10 0x0000000001d269a4 in row_ins_sorted_clust_index_entry (mode=33,
    index=0x7f816cf06720, entry=0x7f816ceed370, n_ext=0, thr=0x7f816cba1f28)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/row/row0ins.cc:2777
#11 0x0000000001d28358 in row_ins_clust_index_entry (index=0x7f816cf06720,
    entry=0x7f816ceed370, thr=0x7f816cba1f28, n_ext=0, dup_chk_only=false)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/row/row0ins.cc:3326
#12 0x0000000001d47116 in row_insert_for_mysql_using_cursor (
    mysql_rec=0x7f816ceb34b0 "\001", prebuilt=0x7f816cf54e40)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/row/row0mysql.cc:1604
#13 0x0000000001d47d1a in row_insert_for_mysql (
    mysql_rec=0x7f816ceb34b0 "\001", prebuilt=0x7f816cf54e40)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/row/row0mysql.cc:1864
#14 0x0000000001beeac5 in ha_innobase::intrinsic_table_write_row (
    this=0x7f816cba59d8, record=0x7f816ceb34b0 "\001")
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:7439
#15 0x0000000001beeb8c in ha_innobase::write_row (this=0x7f816cba59d8,
    record=0x7f816ceb34b0 "\001")
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:7463
#16 0x0000000000f8967a in handler::ha_write_row (this=0x7f816cba59d8,
    buf=0x7f816ceb34b0 "\001")
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/hand--Type <RET> for more, q to quit, c to continue without paging--
ler.cc:8093
#17 0x000000000162299c in schema_table_store_record (thd=0x7f816cc23130,
    table=0x7f816cb50270)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:3205
#18 0x0000000001c4fa49 in i_s_innodb_buffer_page_fill (thd=0x7f816cc23130,
    tables=0x7f816d150da8, info_array=0x7f816d099af0, num_page=8192)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/i_s.cc:5392
#19 0x0000000001c501c1 in i_s_innodb_fill_buffer_pool (thd=0x7f816cc23130,
    tables=0x7f816d150da8, buf_pool=0x5504e38, pool_id=0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/i_s.cc:5616
#20 0x0000000001c5035d in i_s_innodb_buffer_page_fill_table (
    thd=0x7f816cc23130, tables=0x7f816d150da8)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/i_s.cc:5665
#21 0x0000000001634b2e in do_fill_table (thd=0x7f816cc23130,
    table_list=0x7f816d150da8, qep_tab=0x7f816ce19290)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:8189
#22 0x0000000001635038 in get_schema_tables_result (join=0x7f816ce18c90,
    executed_place=PROCESSED_BY_JOIN_EXEC)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:8319
#23 0x000000000160c2a5 in JOIN::prepare_result (this=0x7f816ce18c90)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_select.cc:915
#24 0x000000000156e54d in JOIN::exec (this=0x7f816ce18c90)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:131
#25 0x000000000160ae6a in handle_query (thd=0x7f816cc23130,
    lex=0x7f816cc252c8, result=0x7f816d16c1e0, added_options=0,
    removed_options=0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_select.cc:191
#26 0x00000000015bfd1a in execute_sqlcom_select (thd=0x7f816cc23130,
--Type <RET> for more, q to quit, c to continue without paging--
``` 

断点: i_s_innodb_set_page_type

  - 通过SQL: select table_name, FIX_COUNT, count(1) from information_schema.INNODB_BUFFER_PAGE group by table_name, fix_count; 触发
  - 查看bpage的index_id

发现: intrinsic_table 的数据页, index_id = 1, 会计入 SYS_TABLES 

  - intrinsic table: An intrinsic table is a special kind of temporary table that is invisible to the end user

下一步: 

  - 也有其他可能, 生成的数据页的index_id = 1
    - index的id生成函数为: dict_build_index_def
    - intrinsic table的第一个index, ID会设置为1, 除此之外, 认为不存在其他可能
  - 排查产生intrinsic table的地方, 列举intrinsic table的使用场景
    - 现场现象1: INNODB_TEMP_TABLE_INFO 表里没有内容
      - INNODB_TEMP_TABLE_INFO 的数据来源:

        - 函数: i_s_innodb_temp_table_info_fill_table

        - 来源: dict_sys->table_non_LRU

    - 现场现象2: 锁定页 不影响redo log刷盘
      - 可能是禁用了redo log, 例如: dict_disable_redo_if_temporary
      - 标志位: MTR_LOG_NO_REDO
        - mtr没有封闭
          - row_update_inplace_for_intrinsic
            - 找到 bug: <https://github.com/mysql/mysql-server/commit/505521556d3fa01e2107b2190594dcb8c413b809>
            - 在之后章节分析
          - row_search_no_mvcc
          - PageBulk::init
          - PageBulk::latch 中, mtr没有封闭
          - IndexPurge::open
          - IndexPurge::next
          - IndexPurge::purge
          - row_ins_sorted_clust_index_entry
          - IndexIterator.search
          - row_upd_clust_step: 排除
        - dict_disable_redo_if_temporary
          - ??
    - 需要找到: 在 INNODB_TEMP_TABLE_INFO 不出现的 intrinsic table

标记intrinsic table的堆栈: 

```
(gdb) bt
#0  create_table_info_t::innobase_table_flags (this=0x7f81ac4492d0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:11552
#1  0x0000000001bf6896 in create_table_info_t::prepare_create_table (
    this=0x7f81ac4492d0,
    name=0x7f816cb51aa8 "/root/sandboxes/msb_5_7_29/tmp/#sql_33e4_0")
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:11859
#2  0x0000000001bf75b9 in ha_innobase::create (this=0x7f816cba59d8,
    name=0x7f816cb51aa8 "/root/sandboxes/msb_5_7_29/tmp/#sql_33e4_0",
    form=0x7f81ac449540, create_info=0x7f81ac4493b0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:12223
#3  0x0000000001666182 in create_innodb_tmp_table (table=0x7f81ac449540,
    keyinfo=0x0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_tmp_table.cc:2275
#4  0x0000000001666fba in create_ondisk_from_heap (thd=0x7f816cc23130,
    table=0x7f816cb50270, start_recinfo=0x7f816cb51268,
    recinfo=0x7f816d16b818, error=135, ignore_last_dup=false,
    is_duplicate=0x0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_tmp_table.cc:2546
#5  0x00000000016229f7 in schema_table_store_record (thd=0x7f816cc23130,
    table=0x7f816cb50270)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:3209
#6  0x0000000001c4fa49 in i_s_innodb_buffer_page_fill (thd=0x7f816cc23130,
    tables=0x7f816d150da8, info_array=0x7f816d099af0, num_page=8192)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/i_s.cc:5392
#7  0x0000000001c501c1 in i_s_innodb_fill_buffer_pool (thd=0x7f816cc23130,
    tables=0x7f816d150da8, buf_pool=0x5504e38, pool_id=0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/i_s.cc:5616
#8  0x0000000001c5035d in i_s_innodb_buffer_page_fill_table (
    thd=0x7f816cc23130, tables=0x7f816d150da8)
--Type <RET> for more, q to quit, c to continue without paging--
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/i_s.cc:5665
#9  0x0000000001634b2e in do_fill_table (thd=0x7f816cc23130,
    table_list=0x7f816d150da8, qep_tab=0x7f816ce19290)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:8189
#10 0x0000000001635038 in get_schema_tables_result (join=0x7f816ce18c90,
    executed_place=PROCESSED_BY_JOIN_EXEC)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:8319
#11 0x000000000160c2a5 in JOIN::prepare_result (this=0x7f816ce18c90)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_select.cc:915
#12 0x000000000156e54d in JOIN::exec (this=0x7f816ce18c90)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:131
#13 0x000000000160ae6a in handle_query (thd=0x7f816cc23130,
    lex=0x7f816cc252c8, result=0x7f816d16c1e0, added_options=0,
    removed_options=0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_select.cc:191
#14 0x00000000015bfd1a in execute_sqlcom_select (thd=0x7f816cc23130,
    all_tables=0x7f816d150da8)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:5155
#15 0x00000000015b8af9 in mysql_execute_command (thd=0x7f816cc23130,
    first_level=true)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:2826
#16 0x00000000015c0c25 in mysql_parse (thd=0x7f816cc23130,
    parser_state=0x7f81ac44c670)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:5584
#17 0x00000000015b57a0 in dispatch_command (thd=0x7f816cc23130,
    com_data=0x7f81ac44cde0, command=COM_QUERY)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:1491
--Type <RET> for more, q to quit, c to continue without paging--
#18 0x00000000015b45e5 in do_command (thd=0x7f816cc23130)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:1032
#19 0x00000000016ec5e5 in handle_connection (arg=0x5cbe9a0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/conn_handler/connection_handler_per_thread.cc:313
#20 0x0000000001b43658 in pfs_spawn_thread (arg=0x5d48410)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/perfschema/pfs.cc:2197
#21 0x00007f81cdf4a6db in start_thread ()
   from /lib/x86_64-linux-gnu/libpthread.so.0
#22 0x00007f81cc8e371f in clone () from /lib/x86_64-linux-gnu/libc.so.6
(gdb)
(gdb)
``` 

# 研究 row_update_inplace_for_intrinsic bug

找到 bug: <https://github.com/mysql/mysql-server/commit/505521556d3fa01e2107b2190594dcb8c413b809>

堆栈: 

```
#0  row_update_inplace_for_intrinsic (node=0x7f9834af0948)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/row/row0mysql.cc:2095
#1  0x0000000001d493a8 in row_del_upd_for_mysql_using_cursor (
    mysql_rec=0x7f9834984298 "\213\217", prebuilt=0x7f9834aee250)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/row/row0mysql.cc:2442
#2  0x0000000001d49c75 in row_update_for_mysql (
    mysql_rec=0x7f9834984298 "\213\217", prebuilt=0x7f9834aee250)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/row/row0mysql.cc:2669
#3  0x0000000001bf09d6 in ha_innobase::update_row (this=0x7f9834983a70,
    old_row=0x7f9834984298 "\213\217", new_row=0x7f9834983e80 "\213\217")
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:8257
#4  0x0000000000f89b40 in handler::ha_update_row (this=0x7f9834983a70,
    old_data=0x7f9834984298 "\213\217", new_data=0x7f9834983e80 "\213\217")
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/handler.cc:8134
#5  0x00000000015769d2 in end_update (join=0x7f9834979780,
    qep_tab=0x7f983497de70, end_of_records=false)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:3482
#6  0x0000000001579ce8 in QEP_tmp_table::put_record (this=0x7f983497e2f8,
    end_of_records=false)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:4680
#7  0x000000000157aa69 in QEP_tmp_table::put_record (this=0x7f983497e2f8)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.h:255
#8  0x0000000001570ec3 in sub_select_op (join=0x7f9834979780,
    qep_tab=0x7f983497de70, end_of_records=false)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:1090
#9  0x00000000017f8a0e in JOIN_CACHE::generate_full_extensions (
    this=0x7f983497e160, rec_ptr=0x7f9834b3fff3 "\001\004\217\377\377\001\"")
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_--Type <RET> for more, q to quit, c to continue without paging--
join_buffer.cc:2258
#10 0x00000000017f8fb5 in JOIN_CACHE::join_null_complements (
    this=0x7f983497e160, skip_last=false)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_join_buffer.cc:2419
#11 0x00000000017f8200 in JOIN_CACHE::join_records (this=0x7f983497e160,
    skip_last=false)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_join_buffer.cc:1983
#12 0x00000000017fb657 in JOIN_CACHE::end_send (this=0x7f983497e160)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_join_buffer.h:467
#13 0x0000000001570e18 in sub_select_op (join=0x7f9834979780,
    qep_tab=0x7f983497dcf8, end_of_records=true)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:1076
#14 0x0000000001570f50 in sub_select (join=0x7f9834979780,
    qep_tab=0x7f983497db80, end_of_records=true)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:1233
#15 0x0000000001570f50 in sub_select (join=0x7f9834979780,
    qep_tab=0x7f983497da08, end_of_records=true)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:1233
#16 0x0000000001570aca in do_select (join=0x7f9834979780)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:959
#17 0x000000000156e956 in JOIN::exec (this=0x7f9834979780)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_executor.cc:206
#18 0x000000000160ae6a in handle_query (thd=0x7f9834000e20, lex=
    0x7f9834002fb8, result=0x7f9834978548, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_select.cc:191
#19 0x00000000015bfd1a in execute_sqlcom_select (thd=0x7f9834000e20,
    all_tables=0x7f983400aec0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_--Type <RET> for more, q to quit, c to continue without paging--
parse.cc:5155
#20 0x00000000015b8af9 in mysql_execute_command (thd=0x7f9834000e20,
    first_level=true)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:2826
#21 0x00000000015c0c25 in mysql_parse (thd=0x7f9834000e20,
    parser_state=0x7f988c1d8670)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:5584
#22 0x00000000015b57a0 in dispatch_command (thd=0x7f9834000e20,
    com_data=0x7f988c1d8de0, command=COM_QUERY)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:1491
#23 0x00000000015b45e5 in do_command (thd=0x7f9834000e20)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:1032
#24 0x00000000016ec5e5 in handle_connection (arg=0x47b9eb0)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/conn_handler/connection_handler_per_thread.cc:313
#25 0x0000000001b43658 in pfs_spawn_thread (arg=0x5327e10)
    at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/perfschema/pfs.cc:2197
#26 0x00007f98986b36db in start_thread ()
   from /lib/x86_64-linux-gnu/libpthread.so.0
#27 0x00007f989704c71f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

简化复现场景: 

```
表P在bug测试代码中
 
---
 
SET SESSION big_tables = ON;

SELECT
  table1.col_int_key AS field2,
  MAX(table1.col_int) AS field4
FROM
  P AS table1
WHERE
  table1.col_int_key BETWEEN 7 AND 8
GROUP BY
  field2;
``` 

再次简化: 

```
表内容: 
mysql [localhost:5729] {msandbox} (test) > select * from test.a;
+---+------+------+
| a | b    | c    |
+---+------+------+
| 1 |    1 | NULL |
| 2 |    1 |    2 |
+---+------+------+
2 rows in set (0.01 sec)
 
触发SQL:
select b, max(c) from test.a group by b;
 
穷举时, c列从NULL变成非NULL, 长度变化, 会触发row_upd_changes_field_size_or_external = true, 触发相关bug
``` 

复现: 

  - 构造a表, 三列a/b/c, 行数大于 32768行, 
  - update test.a set c = NULL;
  - update test.a set c=100 where a % 100 =0;
  - SET SESSION big_tables = ON; select b, max(c) from test.a group by b;
  - 核心观点: 
    - 让group by b计算时, 先找到 C=NULL 的记录, 然后在找到 C=100的记录, 就会触发bug逻辑
    - 想看到多个锁定页, 就需要构造的临时表具有多页, 需要多一点的数据, 并且在不同页上触发bug
    - 触发bug的页号越多, 保留页就越多
  - 效果: 
    - ![image2022-4-27 23:35:24.png](/assets/01KJBYJCDGQG0YRF8688HMW9VH/image2022-4-27%2023%3A35%3A24.png)

就此解决

# 整理结论

确认与 <https://github.com/mysql/mysql-server/commit/505521556d3fa01e2107b2190594dcb8c413b809> 有关

触发条件: 

  - select 触发内部临时表的构建 (以group by为例)
  - group by 结果表足够大, 需要使用磁盘临时表
  - 构造 group by 结果表时, 表数据从NULL 变成 非NULL 

复现: 

  - create table aa(aa int, bb int, cc int);
  - insert into aa values(1, NULL, NULL);
  - insert into aa select aa+ (select count(1) from aa), NULL, NULL from aa;
    - 反复执行, 构造aa表超过60000行
  - update test.aa set bb = rand()*30000;
  - update test.aa set cc=100 where aa % 100 =0;
  - SET SESSION big_tables = ON; select bb, max(cc) from test.aa group by bb;
  - (SET SESSION big_tables = ON; 用于保证使用磁盘临时表. 当临时表结果足够大时, 不需要此参数)

效果: 

![image2022-4-27 23:56:14.png](/assets/01KJBYJCDGQG0YRF8688HMW9VH/image2022-4-27%2023%3A56%3A14.png)

修复: 

代码通过另一个问题进行了修复: <https://github.com/mysql/mysql-server/commit/b161d236ede9e087642c35a28818ebe7636c27fc>

修复版本: 5.7.31

# 知识点

  - persistent cursor: <https://zhuanlan.zhihu.com/p/164728032>

# MySQL 8.0.27

改进了innodb_buffer_page的显示, 将临时表的页归属到了 INDEX类别

进行临时表查询时, 通过: select * from innodb_buffer_page order by OLDEST_MODIFICATION desc limit 1\G 查看

![image2022-4-28 15:13:43.png](/assets/01KJBYJCDGQG0YRF8688HMW9VH/image2022-4-28%2015%3A13%3A43.png)
