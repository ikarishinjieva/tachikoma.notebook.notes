---
title: 20211124 - 农行load data引发锁的分析
confluence_page_id: 1573162
created_at: 2021-11-24T14:14:25+00:00
updated_at: 2021-12-08T11:00:46+00:00
---

# 问题描述

如实验2

# 实验1

```
session 1: 
 
session 1> set session transaction isolation level read uncommitted;
session 1> begin;
session 1> insert into a values(10);
 
session 2> set session transaction isolation level read uncommitted;
session 2> begin;
session 2> insert into a values(40);
 
session 1> insert into a values(40);
session 2> insert into a values(10); 
 
引发死锁后, 通过session 3观察
 
session 3> select * from a for share SKIP LOCKED; //将锁显示全
session 3> select * from performance_schema.data_locks;
 
``` 

发现现象: 

```
mysql [localhost:8018] {msandbox} (performance_schema) > select * from data_locks;
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+---------------+-------------+-----------+
| ENGINE | ENGINE_LOCK_ID                        | ENGINE_TRANSACTION_ID | THREAD_ID | EVENT_ID | OBJECT_SCHEMA | OBJECT_NAME | PARTITION_NAME | SUBPARTITION_NAME | INDEX_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+---------------+-------------+-----------+
| INNODB | 140313227750864:1065:140313159077752  |                  2729 |        87 |      153 | test          | a           | NULL           | NULL              | NULL       |       140313159077752 | TABLE     | IX            | GRANTED     | NULL      |
| INNODB | 140313227750864:8:4:2:140313159075216 |                  2729 |       110 |       19 | test          | a           | NULL           | NULL              | PRIMARY    |       140313159075216 | RECORD    | X,REC_NOT_GAP | GRANTED     | 10        |
| INNODB | 140313227750864:8:4:3:140313159075216 |                  2729 |       110 |       19 | test          | a           | NULL           | NULL              | PRIMARY    |       140313159075216 | RECORD    | X,REC_NOT_GAP | GRANTED     | 40        |
| INNODB | 140313227750864:8:4:2:140313159075560 |                  2729 |       102 |      103 | test          | a           | NULL           | NULL              | PRIMARY    |       140313159075560 | RECORD    | S,GAP         | GRANTED     | 10        |
| INNODB | 140313227750864:8:4:3:140313159075560 |                  2729 |       102 |      103 | test          | a           | NULL           | NULL              | PRIMARY    |       140313159075560 | RECORD    | S,GAP         | GRANTED     | 40        |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+---------------+-------------+-----------+
5 rows in set (0.00 sec)
``` 

需要解释: 

  1. 为何 RUC 隔离级别会出现 S,GAP 锁

# 删除行时, 会进行gap锁继承

gdb调试实验1的场景, 发现 GAP锁 是由以下堆栈生成: 

```
#0  lock_rec_add_to_queue (type_mode=547, block=0x7f21d7d62580, heap_no=4, index=0x7f217cb27ff8, trx=0x7f21ee74da68,
    we_own_trx_mutex=false) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1517
#1  0x0000000002006306 in lock_rec_inherit_to_gap (heir_block=heir_block@entry=0x7f21d7d62580,
    block=block@entry=0x7f21d7d62580, heir_heap_no=heir_heap_no@entry=4, heap_no=heap_no@entry=6)
    at ../../../mysql-8.0.18/storage/innobase/include/lock0priv.ic:264
#2  0x0000000002006ae5 in lock_update_delete (block=0x7f21d7d62580, rec=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:3443
#3  0x00000000021b08b9 in btr_cur_optimistic_delete_func (cursor=cursor@entry=0x7f217d499c28,
    mtr=mtr@entry=0x7f21ec0a6c60) at ../../../mysql-8.0.18/storage/innobase/btr/btr0cur.cc:4612
#4  0x00000000023197a5 in row_undo_ins_remove_clust_rec (node=0x7f217d499bb8)
    at ../../../mysql-8.0.18/storage/innobase/row/row0uins.cc:120
#5  0x000000000231b0e2 in row_undo_ins (node=node@entry=0x7f217d499bb8, thr=thr@entry=0x7f217d499498)
    at ../../../mysql-8.0.18/storage/innobase/row/row0uins.cc:495
#6  0x00000000020f41c7 in row_undo (thr=0x7f217d499498, node=0x7f217d499bb8)
    at ../../../mysql-8.0.18/storage/innobase/row/row0undo.cc:295
#7  row_undo_step (thr=thr@entry=0x7f217d499498) at ../../../mysql-8.0.18/storage/innobase/row/row0undo.cc:362
#8  0x0000000002086078 in que_thr_step (thr=0x7f217d499498)
    at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:921
#9  que_run_threads_low (thr=0x7f217d499498) at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:974
#10 que_run_threads (thr=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:1009
#11 0x000000000214e27b in trx_rollback_to_savepoint_low (trx=trx@entry=0x7f21ee74da68, savept=savept@entry=0x0)
    at ../../../mysql-8.0.18/storage/innobase/trx/trx0roll.cc:113
#12 0x000000000214e502 in trx_rollback_to_savepoint (trx=trx@entry=0x7f21ee74da68, savept=savept@entry=0x0)
    at ../../../mysql-8.0.18/storage/innobase/trx/trx0roll.cc:150
#13 0x00000000020be242 in row_mysql_handle_errors (new_err=0x7f21ec0a7784, trx=0x7f21ee74da68, thr=0x7f217cb85740,
    savept=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0mysql.cc:726
#14 0x00000000020bf09d in row_insert_for_mysql_using_ins_graph (mysql_rec=<optimized out>, prebuilt=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0mysql.cc:1593
#15 0x0000000001f95ac0 in ha_innobase::write_row (this=0x7f217c452ac8,
    record=0x7f217c9b2b98 "\377", <incomplete sequence \310>)
    at ../../../mysql-8.0.18/storage/innobase/handler/ha_innodb.cc:8581
#16 0x00000000010c4338 in handler::ha_write_row (this=0x7f217c452ac8,
    buf=0x7f217c9b2b98 "\377", <incomplete sequence \310>) at ../../mysql-8.0.18/sql/handler.cc:7740
#17 0x00000000012a7f05 in write_record (thd=0x7f217c6c3e40, table=0x7f217ca731c0, info=0x7f21ec0a7d50,
    update=0x7f21ec0a7dd0) at ../../mysql-8.0.18/sql/sql_insert.cc:1946
#18 0x00000000012a8f29 in Sql_cmd_insert_values::execute_inner (this=0x7f217d4c1980, thd=0x7f217c6c3e40)
    at ../../mysql-8.0.18/sql/sql_insert.cc:614
#19 0x0000000000ed98b8 in Sql_cmd_dml::execute (this=0x7f217d4c1980, thd=0x7f217c6c3e40)
    at ../../mysql-8.0.18/sql/sql_select.cc:704
#20 0x0000000000e8cbab in mysql_execute_command (thd=0x7f217c6c3e40, first_level=<optimized out>)
    at ../../mysql-8.0.18/sql/sql_parse.cc:3450
#21 0x0000000000e8e8b7 in mysql_parse (thd=thd@entry=0x7f217c6c3e40, parser_state=parser_state@entry=0x7f21ec0a95f0)
    at ../../mysql-8.0.18/sql/sql_parse.cc:5257
#22 0x0000000000e91244 in dispatch_command (thd=0x7f217c6c3e40, com_data=<optimized out>, command=COM_QUERY)
    at ../../mysql-8.0.18/sql/sql_parse.cc:1765
#23 0x0000000000e91cb4 in do_command (thd=thd@entry=0x7f217c6c3e40) at ../../mysql-8.0.18/sql/sql_parse.cc:1273
#24 0x0000000000fa4c78 in handle_connection (arg=arg@entry=0x491b800)
    at ../../mysql-8.0.18/sql/conn_handler/connection_handler_per_thread.cc:302
#25 0x00000000023b04ac in pfs_spawn_thread (arg=0x4776fa0) at ../../../mysql-8.0.18/storage/perfschema/pfs.cc:2854
#26 0x00007f22031696db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#27 0x00007f22011d871f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

删除行时, 会将该行相关的锁 转移到 其后侧的行, 用以保持语义. 

举例: 

```
隔离级别 RR,  插入数据 (50), (100)
 
事务A: delete from a where a < 60, 不提交
锁状况:
mysql [localhost:8018] {msandbox} ((none)) > select * from performance_schema.data_locks;
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+-----------+
| ENGINE | ENGINE_LOCK_ID                        | ENGINE_TRANSACTION_ID | THREAD_ID | EVENT_ID | OBJECT_SCHEMA | OBJECT_NAME | PARTITION_NAME | SUBPARTITION_NAME | INDEX_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE | LOCK_MODE | LOCK_STATUS | LOCK_DATA |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+-----------+
| INNODB | 140556904483432:1065:140556831357096  |                  4222 |        53 |       60 | test          | a           | NULL           | NULL              | NULL       |       140556831357096 | TABLE     | IX        | GRANTED     | NULL      |
| INNODB | 140556904483432:8:4:3:140556831354168 |                  4222 |        53 |       60 | test          | a           | NULL           | NULL              | PRIMARY    |       140556831354168 | RECORD    | X         | GRANTED     | 50        |
| INNODB | 140556904483432:8:4:2:140556831354512 |                  4222 |        53 |       60 | test          | a           | NULL           | NULL              | PRIMARY    |       140556831354512 | RECORD    | X,GAP     | GRANTED     | 100       |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+-----------+
3 rows in set (0.00 sec)
 
事务B: 删除 (100), 并提交
再观察锁:
mysql [localhost:8018] {msandbox} ((none)) > select * from performance_schema.data_locks;
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+------------------------+
| ENGINE | ENGINE_LOCK_ID                        | ENGINE_TRANSACTION_ID | THREAD_ID | EVENT_ID | OBJECT_SCHEMA | OBJECT_NAME | PARTITION_NAME | SUBPARTITION_NAME | INDEX_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE | LOCK_MODE | LOCK_STATUS | LOCK_DATA              |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+------------------------+
| INNODB | 140556904483432:1065:140556831357096  |                  4222 |        53 |       60 | test          | a           | NULL           | NULL              | NULL       |       140556831357096 | TABLE     | IX        | GRANTED     | NULL                   |
| INNODB | 140556904483432:8:4:1:140556831354168 |                  4222 |        53 |       60 | test          | a           | NULL           | NULL              | PRIMARY    |       140556831354168 | RECORD    | X         | GRANTED     | supremum pseudo-record |
| INNODB | 140556904483432:8:4:3:140556831354168 |                  4222 |        53 |       60 | test          | a           | NULL           | NULL              | PRIMARY    |       140556831354168 | RECORD    | X         | GRANTED     | 50                     |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+------------------------+
3 rows in set (0.00 sec) 
 
发现锁转移
``` 

是否发生锁转移的判断条件为: 

![image2021-11-26 14:49:51.png](/assets/01KJBYFD3950T3XNVP5R4BXYZV/image2021-11-26%2014%3A49%3A51.png)

该场景中, 主要变量是 inherit_all, 意思是该锁因为某条件, 被标记成必须要转移的锁

inherit_all 在 lock_protect_locks_till_statement_end 方法中被标记, 被以下两个代码点调用: 

  - lock_sec_rec_read_check_and_lock
  - lock_clust_rec_read_check_and_lock

带入实验1的死锁场景, 调用lock_protect_locks_till_statement_end的堆栈为: 

```
#0  lock_protect_locks_till_statement_end (thr=0x7fd578098e80)
    at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:2565
#1  lock_clust_rec_read_check_and_lock (thr=0x7fd578098e80, gap_mode=1024,
    mode=LOCK_S, sel_mode=SELECT_ORDINARY, offsets=0x0, index=0x7fd5a8033378,
    rec=<optimized out>, block=0x7fd5dfd68100,
    duration=lock_duration_t::AT_LEAST_STATEMENT)
    at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:5957
#2  lock_clust_rec_read_check_and_lock (
    duration=duration@entry=lock_duration_t::AT_LEAST_STATEMENT, block=0x7fd5dfd68100,
    rec=<optimized out>, index=0x7fd5a8033378, offsets=<optimized out>,
    sel_mode=sel_mode@entry=SELECT_ORDINARY, mode=LOCK_S, gap_mode=1024,
    thr=0x7fd578098e80)
    at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:5929
#3  0x00000000020a42e2 in row_ins_set_rec_lock (thr=0x7fd578098e80,
    offsets=<optimized out>, index=<optimized out>, rec=<optimized out>,
    block=<optimized out>, type=<optimized out>, mode=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:1317
#4  row_ins_set_rec_lock (mode=<optimized out>, type=<optimized out>,
    block=<optimized out>, rec=<optimized out>, index=<optimized out>,
    offsets=<optimized out>, thr=0x7fd578098e80)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:1308
#5  0x00000000020a8398 in row_ins_duplicate_error_in_clust (mtr=0x7fd5ec0cd120,
    thr=0x7fd578098e80, entry=0x7fd5780b6c58, cursor=0x7fd5ec0cc9e0, flags=0)
    at ../../../mysql-8.0.18/storage/innobase/include/row0mysql.h:900
#6  row_ins_clust_index_entry_low (flags=<optimized out>, mode=2,
    index=0x7fd5a8033378, n_uniq=1, entry=<optimized out>, n_ext=<optimized out>,
    thr=<optimized out>, dup_chk_only=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:2467
#7  0x00000000020acd08 in row_ins_clust_index_entry (index=0x7fd5a8033378,
    entry=0x7fd5780b6c58, thr=0x7fd578098e80, n_ext=0, dup_chk_only=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3102
#8  0x00000000020ada74 in row_ins_index_entry (thr=0x7fd578098e80,
    multi_val_pos=@0x7fd578098cf8: 0, entry=<optimized out>, index=0x7fd5a8033378)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3295
#9  row_ins_index_entry_step (thr=0x7fd578098e80, node=0x7fd578098c40)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3432
#10 row_ins (thr=0x7fd578098e80, node=0x7fd578098c40)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3551
#11 row_ins_step (thr=thr@entry=0x7fd578098e80)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3675
#12 0x00000000020bf0c4 in row_insert_for_mysql_using_ins_graph (
--Type <RET> for more, q to quit, c to continue without paging--
    mysql_rec=<optimized out>, prebuilt=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0mysql.cc:1580
#13 0x0000000001f95ac0 in ha_innobase::write_row (this=0x7fd57807aa28,
    record=0x7fd578097f28 "\377\n")
    at ../../../mysql-8.0.18/storage/innobase/handler/ha_innodb.cc:8581
#14 0x00000000010c4338 in handler::ha_write_row (this=0x7fd57807aa28,
    buf=0x7fd578097f28 "\377\n") at ../../mysql-8.0.18/sql/handler.cc:7740
#15 0x00000000012a7f05 in write_record (thd=0x7fd578000e20, table=0x7fd5780161c0,
    info=0x7fd5ec0cdd50, update=0x7fd5ec0cddd0)
    at ../../mysql-8.0.18/sql/sql_insert.cc:1946
#16 0x00000000012a8f29 in Sql_cmd_insert_values::execute_inner (this=0x7fd578015140,
    thd=0x7fd578000e20) at ../../mysql-8.0.18/sql/sql_insert.cc:614
#17 0x0000000000ed98b8 in Sql_cmd_dml::execute (this=0x7fd578015140,
    thd=0x7fd578000e20) at ../../mysql-8.0.18/sql/sql_select.cc:704
#18 0x0000000000e8cbab in mysql_execute_command (thd=0x7fd578000e20,
    first_level=<optimized out>) at ../../mysql-8.0.18/sql/sql_parse.cc:3450
#19 0x0000000000e8e8b7 in mysql_parse (thd=thd@entry=0x7fd578000e20,
    parser_state=parser_state@entry=0x7fd5ec0cf5f0)
    at ../../mysql-8.0.18/sql/sql_parse.cc:5257
#20 0x0000000000e91244 in dispatch_command (thd=0x7fd578000e20,
    com_data=<optimized out>, command=COM_QUERY)
    at ../../mysql-8.0.18/sql/sql_parse.cc:1765
#21 0x0000000000e91cb4 in do_command (thd=thd@entry=0x7fd578000e20)
    at ../../mysql-8.0.18/sql/sql_parse.cc:1273
#22 0x0000000000fa4c78 in handle_connection (arg=arg@entry=0x52eb170)
    at ../../mysql-8.0.18/sql/conn_handler/connection_handler_per_thread.cc:302
#23 0x00000000023b04ac in pfs_spawn_thread (arg=0x52d4680)
    at ../../../mysql-8.0.18/storage/perfschema/pfs.cc:2854
#24 0x00007fd608fe86db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#25 0x00007fd60705771f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

在insert的过程中, row_ins_duplicate_error_in_clust 检查在记录重复时, 会将记录上的锁标记为 需要转移

改造实验1, 不触发死锁:

```
session 1: 
 
session 1> set session transaction isolation level read uncommitted;
session 1> begin;
session 1> insert into a values(10);
 
session 2> set session transaction isolation level read uncommitted;
session 2> begin;
session 2> insert into a values(40);
 
session 1> insert into a values(40);
会触发 lock_protect_locks_till_statement_end
session 1会等待锁
session 2> rollback;
rollback 后, session 1会继续
 

session 3> select * from performance_schema.data_locks;
会出现S, GAP锁
mysql [localhost:8018] {msandbox} (performance_schema) > select * from data_locks;
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+-----------+
| ENGINE | ENGINE_LOCK_ID                        | ENGINE_TRANSACTION_ID | THREAD_ID | EVENT_ID | OBJECT_SCHEMA | OBJECT_NAME | PARTITION_NAME | SUBPARTITION_NAME | INDEX_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE | LOCK_MODE | LOCK_STATUS | LOCK_DATA |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+-----------+
| INNODB | 140556904483432:1065:140556831357096  |                  4355 |        62 |       30 | test          | a           | NULL           | NULL              | NULL       |       140556831357096 | TABLE     | IX        | GRANTED     | NULL      |
| INNODB | 140556904483432:8:4:4:140556831354856 |                  4355 |        63 |       31 | test          | a           | NULL           | NULL              | PRIMARY    |       140556831354856 | RECORD    | S,GAP     | GRANTED     | 40        |
| INNODB | 140556904483432:8:4:5:140556831354856 |                  4355 |        63 |       31 | test          | a           | NULL           | NULL              | PRIMARY    |       140556831354856 | RECORD    | S,GAP     | GRANTED     | 60        |
+--------+---------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+-----------+-------------+-----------+
3 rows in set (0.00 sec)
``` 

# 实验2 - 模拟用户场景

```
~/sandboxes/msb_8_0_18/use -e 'delete from test.b'; /tmp/mysql-shell-8.0.25-linux-glibc2.12-x86-64bit/bin/mysqlsh msandbox:msandbox@127.0.0.1:8018/test --ssl-mode=DISABLED -- util import-table /tmp/sbtest1.txt --schema=test --table=b --bytes-per-chunk=1689030  --threads=6
``` 

联合使用 delete 和 mysqlsh, 可重现死锁

调用lock_rec_inherit_to_gap的堆栈: 

```
#0  lock_rec_inherit_to_gap (
    heir_block=heir_block@entry=0x7faeaffb4d00,
    block=block@entry=0x7faeaffb4d00,
    heir_heap_no=heir_heap_no@entry=92, heap_no=heap_no@entry=634)
    at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:2615
#1  0x0000000002006ae5 in lock_update_delete (block=0x7faeaffb4d00,
    rec=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:3443
#2  0x00000000021b08b9 in btr_cur_optimistic_delete_func (
    cursor=cursor@entry=0x7faec04964d8,
    mtr=mtr@entry=0x7fae997f9560)
    at ../../../mysql-8.0.18/storage/innobase/btr/btr0cur.cc:4612
#3  0x00000000020de06b in row_purge_remove_clust_if_poss_low (
    node=0x7faec0496428, mode=2)
    at ../../../mysql-8.0.18/storage/innobase/include/btr0pcur.h:719
#4  0x00000000020e0ba7 in row_purge_remove_clust_if_poss (
    node=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:228
#5  row_purge_del_mark (node=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:671
#6  row_purge_record_func (thd=0x7fae6c000b20,
    updated_extern=<optimized out>,
    undo_rec=0x7faec049a9f8 "\006\251N", node=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:1081
#7  row_purge (thr=<optimized out>, undo_rec=<optimized out>,
    node=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:1155
#8  row_purge_step (thr=thr@entry=0x7faec03635a8)
    at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:1227
#9  0x0000000002086128 in que_thr_step (thr=0x7faec03635a8)
    at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:923
#10 que_run_threads_low (thr=0x7faec03635a8)
    at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:974
#11 que_run_threads (thr=thr@entry=0x7faec03635a8)
    at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:1009
#12 0x00000000021150dc in srv_task_execute ()
    at ../../../mysql-8.0.18/storage/innobase/srv/srv0srv.cc:2724
#13 srv_worker_thread ()
    at ../../../mysql-8.0.18/storage/innobase/srv/srv0srv.cc:2763
#14 0x000000000202eac5 in std::__invoke_impl<void, void (*&)()> (
    __f=<synthetic pointer>: <optimized out>)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:59
#15 std::__invoke<void (*&)()> (
    __fn=<synthetic pointer>: <optimized out>)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#16 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:400
#17 std::_Bind<void (*())()>::operator()<, void>() (
    this=<synthetic pointer>)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:484
#18 Runnable::operator()<void (*)()> (
    f=@0x58eab18: 0x2114ce0 <srv_worker_thread()>, this=0x58eab20)
    at ../../../mysql-8.0.18/storage/innobase/include/os0thread-create.h:101
#19 std::__invoke_impl<void, Runnable, void (*)()> (__f=...)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:60
#20 std::__invoke<Runnable, void (*)()> (__fn=...)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x58eab18)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:244
#22 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x58eab18)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:253
#23 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x58eab10)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:196
#24 0x00000000025d3f0f in execute_native_thread_routine ()
#25 0x00007faedb56d6db in start_thread ()
   from /lib/x86_64-linux-gnu/libpthread.so.0
#26 0x00007faed95dc71f in clone ()
   from /lib/x86_64-linux-gnu/libc.so.6
``` 

调用row_ins_duplicate_error_in_clust的堆栈: 

```
#0  row_ins_duplicate_error_in_clust (mtr=0x7faea83a6bf0, thr=0x7fae1c01a700,
    entry=0x7fae1c0136b8, cursor=0x7faea83a64b0, flags=0)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:2104
#1  row_ins_clust_index_entry_low (flags=<optimized out>, mode=2, index=0x7fae2000f1c8,
    n_uniq=1, entry=<optimized out>, n_ext=<optimized out>, thr=<optimized out>,
    dup_chk_only=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:2467
#2  0x00000000020acd08 in row_ins_clust_index_entry (index=0x7fae2000f1c8,
    entry=0x7fae1c0136b8, thr=0x7fae1c01a700, n_ext=0, dup_chk_only=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3102
#3  0x00000000020ada74 in row_ins_index_entry (thr=0x7fae1c01a700,
    multi_val_pos=@0x7fae1c01a578: 0, entry=<optimized out>, index=0x7fae2000f1c8)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3295
#4  row_ins_index_entry_step (thr=0x7fae1c01a700, node=0x7fae1c01a4c0)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3432
#5  row_ins (thr=0x7fae1c01a700, node=0x7fae1c01a4c0)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3551
#6  row_ins_step (thr=thr@entry=0x7fae1c01a700)
    at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3675
#7  0x00000000020bf0c4 in row_insert_for_mysql_using_ins_graph (mysql_rec=<optimized out>,
    prebuilt=<optimized out>)
    at ../../../mysql-8.0.18/storage/innobase/row/row0mysql.cc:1580
#8  0x0000000001f95ac0 in ha_innobase::write_row (this=0x7fae1c0180d8,
    record=0x7fae1c0197a8 "\377A\271t")
    at ../../../mysql-8.0.18/storage/innobase/handler/ha_innodb.cc:8581
#9  0x00000000010c4338 in handler::ha_write_row (this=0x7fae1c0180d8,
    buf=0x7fae1c0197a8 "\377A\271t") at ../../mysql-8.0.18/sql/handler.cc:7740
#10 0x00000000012a7f05 in write_record (thd=thd@entry=0x7fae20000dd0,
    table=table@entry=0x7fae1c017730, info=info@entry=0x7faea83a7940,
    update=update@entry=0x0) at ../../mysql-8.0.18/sql/sql_insert.cc:1946
#11 0x00000000012b0e65 in Sql_cmd_load_table::read_sep_field (this=0x7fae20014a80,
    thd=0x7fae20000dd0, info=..., table_list=0x7fae20014b90, read_info=..., enclosed=...,
    skip_lines=0) at ../../mysql-8.0.18/sql/sql_load.cc:1043
#12 0x00000000012b467c in Sql_cmd_load_table::execute_inner (this=<optimized out>,
    thd=0x7fae20000dd0, handle_duplicates=DUP_ERROR)
    at ../../mysql-8.0.18/sql/sql_load.cc:542
#13 0x00000000012b4c63 in Sql_cmd_load_table::execute (this=0x7fae20014a80,
    thd=0x7fae20000dd0) at ../../mysql-8.0.18/sql/sql_load.cc:2000
#14 0x0000000000e8cbab in mysql_execute_command (thd=0x7fae20000dd0,
    first_level=<optimized out>) at ../../mysql-8.0.18/sql/sql_parse.cc:3450
#15 0x0000000000e8e8b7 in mysql_parse (thd=thd@entry=0x7fae20000dd0,
    parser_state=parser_state@entry=0x7faea83a95f0)
    at ../../mysql-8.0.18/sql/sql_parse.cc:5257
#16 0x0000000000e91244 in dispatch_command (thd=0x7fae20000dd0, com_data=<optimized out>,
    command=COM_QUERY) at ../../mysql-8.0.18/sql/sql_parse.cc:1765
#17 0x0000000000e91cb4 in do_command (thd=thd@entry=0x7fae20000dd0)
    at ../../mysql-8.0.18/sql/sql_parse.cc:1273
#18 0x0000000000fa4c78 in handle_connection (arg=arg@entry=0x59079a0)
    at ../../mysql-8.0.18/sql/conn_handler/connection_handler_per_thread.cc:302
#19 0x00000000023b04ac in pfs_spawn_thread (arg=0x58939a0)
    at ../../../mysql-8.0.18/storage/perfschema/pfs.cc:2854
#20 0x00007faedb56d6db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#21 0x00007faed95dc71f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

用gdb找到RecLock的创建处: 

```
准备gdb.init文件: 
set pagination off
set logging file gdb.txt
set logging on

br RecLock
commands
    bt
    continue
end

continue
 
运行gdb: gdb -p $(pgrep mysqld$) -x gdb.init 2>&1 > gdb.txt
``` 

观察gdb.txt, 发现几种GAP锁的mode: 

```
mode=2563
mode=2851
mode=546
``` 

需要调查这几种mode的来源

临时结论

  1. delete操作后, 才会异步 实际删除记录. 实际删除记录时, 会涉及到将删除的行带有的锁, 让步给右侧记录
  2. 让步的过程中 (lock_rec_inherit_to_gap), 会进行gap锁的继承/转移. 但在此处, MySQL 会有误判. (举例: RUC下, insert的死锁, 会产生S锁)
  3. MySQL的误判的原因, 参考: lock_rec_inherit_to_gap
  4. insert会使用 隐式锁 (p_s看不到), 在一定条件下, 才转换成显式锁. 查看p_s, 可以使用 select * from test.a for share skip locked; select * from performance_schema.data_locks;

# 通用的诊断方法

gdb.init文件: 

```
set pagination off
set logging file gdb.txt
set logging on

br RecLock
commands
    bt
    continue
end

br lock0lock.cc:6694
commands
    p blocking_lock
    p *blocking_lock
    continue
end

br RecLock::lock_add
commands
    p lock
    frame 3
    info locals
    bt
    continue
end

br lock_protect_locks_till_statement_end
commands
    p thr->graph->trx
    p thr->graph->trx->lock
    bt
    continue
end

continue
``` 

其中: 

  1. RecLock, 监听了RecLock锁创建的事件
  2. RecLock::lock_add, 监听了RecLock创建lock_t结构的事件
  3. [lock0lock.cc](<http://lock0lock.cc>):6694, 为Deadlock_notifier::notify, 在打印deadlock的地方, 可以获得hold lock的地址和信息
  4. lock_protect_locks_till_statement_end, 用于获取标记trx为inherit_all的位置, 此处非通用方法, 可以忽略

使用 gdb -p $(pgrep mysqld$) -x gdb.init 2>&1 > gdb.txt, 调试MySQL, 重现实验2

分析日志: 

```
MySQL 死锁信息:
------------------------
LATEST DETECTED DEADLOCK
------------------------
2021-12-01 09:28:05 0x7f815fcf9700
*** (1) TRANSACTION:
TRANSACTION 9077, ACTIVE 203 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 205 lock struct(s), heap size 57552, 10318 row lock(s), undo log entries 14979
MySQL thread id 180, OS thread handle 140193571673856, query id 925 localhost 127.0.0.1 msandbox executing
LOAD DATA LOCAL INFILE '/tmp/sbtest1.trim.100000.txt' INTO TABLE `test`.`b` FIELDS TERMINATED BY '	' ESCAPED BY '\\' LINES STARTING BY '' TERMINATED BY '\n'

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 18 page no 1104 n bits 792 index PRIMARY of table `test`.`b` trx id 9077 lock mode S
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 18 page no 1498 n bits 432 index PRIMARY of table `test`.`b` trx id 9077 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 26 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8033665c; asc  3f\;;
 1: len 6; hex 000000002373; asc     #s;;
 2: len 7; hex 01000001692660; asc     i&`;;

*** (2) TRANSACTION:
TRANSACTION 9075, ACTIVE 203 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 199 lock struct(s), heap size 57552, 8191 row lock(s), undo log entries 15134
MySQL thread id 178, OS thread handle 140193442760448, query id 905 localhost 127.0.0.1 msandbox executing
LOAD DATA LOCAL INFILE '/tmp/sbtest1.trim.100000.txt' INTO TABLE `test`.`b` FIELDS TERMINATED BY '	' ESCAPED BY '\\' LINES STARTING BY '' TERMINATED BY '\n'

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 18 page no 1498 n bits 432 index PRIMARY of table `test`.`b` trx id 9075 lock mode S locks gap before rec
Record lock, heap no 26 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8033665c; asc  3f\;;
 1: len 6; hex 000000002373; asc     #s;;
 2: len 7; hex 01000001692660; asc     i&`;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 18 page no 1104 n bits 792 index PRIMARY of table `test`.`b` trx id 9075 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** WE ROLL BACK TRANSACTION (1)

``` 

gdb日志: 

```
Deadlock信息: 
 
Thread 19 "mysqld" hit Breakpoint 2, Deadlock_notifier::notify (trxs_on_cycle=..., victim_trx=0x7f8178d7db70) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:6694
6694    in ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc
$1836 = (const ib_lock_t *) 0x7f816b6908f0
$1837 = {trx = 0x7f8178d7db70, trx_locks = {prev = 0x7f816b690860, next = 0x7f816b6909b0}, index = 0x7f81200336f8, hash = 0x7f8165ee8fa8, {tab_lock = {table = 0x45000000012, locks = {prev = 0x320d50300000318, next = 0x7f8178d7d138}}, rec_lock = {space = 18, page_no = 11
04, n_bits = 792}}, m_psi_internal_thread_id = 219, m_psi_event_id = 8, type_mode = 34}

Thread 19 "mysqld" hit Breakpoint 2, Deadlock_notifier::notify (trxs_on_cycle=..., victim_trx=0x7f8178d7db70) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:6694
6694    in ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc
$1838 = (const ib_lock_t *) 0x7f8165ee8f18
$1839 = {trx = 0x7f8178d7cdd0, trx_locks = {prev = 0x7f8165ee8e88, next = 0x7f8165ee8fa8}, index = 0x7f81200336f8, hash = 0x7f816b691298, {tab_lock = {table = 0x5da00000012, locks = {prev = 0x47800446000001b0, next = 0x4780040008007ee9}}, rec_lock = {space = 18, page_no
 = 1498, n_bits = 432}}, m_psi_internal_thread_id = 218, m_psi_event_id = 8, type_mode = 546}

``` 

以 0x7f8165ee8f18 为例, 查询日志, 获得: 

```
Thread 46 "mysqld" hit Breakpoint 3, RecLock::lock_add (this=0x7f81545abf30, lock=0x7f8165ee8f18, add_to_hash=true) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1254
1254    ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc: No such file or directory.
$1833 = (ib_lock_t *) 0x7f8165ee8f18
#3  0x000000000200557e in lock_move_rec_list_end (new_block=0x7f8163dc9780, block=<optimized out>, rec=0x7f8166004a1d "\200\063^\t") at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:2957
2957    in ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc
rec1_heap_no = <optimized out>
rec2_heap_no = <optimized out>
rec1 = 0x7f8166004093 "\200\063g\256"
rec2 = 0x7f81661942a3 "\200\063g\256"
type_mode = 546
lock = 0x7f8165f0b740
comp = <optimized out>
#0  RecLock::lock_add (this=0x7f81545abf30, lock=0x7f8165ee8f18, add_to_hash=true) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1254
#1  0x0000000002004559 in RecLock::create (this=this@entry=0x7f81545abf30, trx=trx@entry=0x7f8178d7cdd0, add_to_hash=add_to_hash@entry=true, prdt=prdt@entry=0x0) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1329
#2  0x0000000002004c9c in lock_rec_add_to_queue (type_mode=<optimized out>, block=0x7f8163dc9780, heap_no=<optimized out>, index=<optimized out>, trx=0x7f8178d7cdd0, we_own_trx_mutex=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1551
#3  0x000000000200557e in lock_move_rec_list_end (new_block=0x7f8163dc9780, block=<optimized out>, rec=0x7f8166004a1d "\200\063^\t") at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:2957
#4  0x000000000206d91a in page_copy_rec_list_end (new_block=0x7f8163dc9780, block=0x7f8163dc0180, rec=0x7f8166004a1d "\200\063^\t", index=0x7f81200336f8, mtr=0x7f81545acbf0) at ../../../mysql-8.0.18/storage/innobase/page/page0page.cc:644
#5  0x0000000002075599 in page_move_rec_list_end (new_block=new_block@entry=0x7f8163dc9780, block=block@entry=0x7f8163dc0180, split_rec=split_rec@entry=0x7f8166004a1d "\200\063^\t", index=0x7f81200336f8, mtr=mtr@entry=0x7f81545acbf0) at ../../../mysql-8.0.18/storage/innobase/page/page0page.cc:1180
#6  0x00000000021a6d4f in btr_page_split_and_insert (flags=0, cursor=<optimized out>, offsets=0x7f81545ac458, heap=0x7f81545ac450, tuple=0x7f8100008c28, n_ext=0, mtr=0x7f81545acbf0) at ../../../mysql-8.0.18/storage/innobase/btr/btr0btr.cc:2547
#7  0x00000000021adaaa in btr_cur_pessimistic_insert (flags=flags@entry=0, cursor=cursor@entry=0x7f81545ac4b0, offsets=offsets@entry=0x7f81545ac458, heap=heap@entry=0x7f81545ac450, entry=entry@entry=0x7f8100008c28, rec=rec@entry=0x7f81545ac8d0, big_rec=0x7f81545ac448, n_ext=<optimized out>, thr=0x7f8104008348, mtr=0x7f81545acbf0) at ../../../mysql-8.0.18/storage/innobase/btr/btr0cur.cc:3041
#8  0x00000000020a8230 in row_ins_clust_index_entry_low (flags=<optimized out>, mode=<optimized out>, index=0x7f81200336f8, n_uniq=<optimized out>, entry=<optimized out>, n_ext=<optimized out>, thr=<optimized out>, dup_chk_only=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:2530
#9  0x00000000020acd73 in row_ins_clust_index_entry (index=0x7f81200336f8, entry=0x7f8100008c28, thr=0x7f8104008348, n_ext=0, dup_chk_only=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3128
#10 0x00000000020ada74 in row_ins_index_entry (thr=0x7f8104008348, multi_val_pos=@0x7f81040081c0: 0, entry=<optimized out>, index=0x7f81200336f8) at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3295
#11 row_ins_index_entry_step (thr=0x7f8104008348, node=0x7f8104008108) at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3432
#12 row_ins (thr=0x7f8104008348, node=0x7f8104008108) at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3551
#13 row_ins_step (thr=thr@entry=0x7f8104008348) at ../../../mysql-8.0.18/storage/innobase/row/row0ins.cc:3675
#14 0x00000000020bf0c4 in row_insert_for_mysql_using_ins_graph (mysql_rec=<optimized out>, prebuilt=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0mysql.cc:1580
#15 0x0000000001f95ac0 in ha_innobase::write_row (this=0x7f8104011d28, record=0x7f81040133f8 "\377\212\230\063") at ../../../mysql-8.0.18/storage/innobase/handler/ha_innodb.cc:8581
#16 0x00000000010c4338 in handler::ha_write_row (this=0x7f8104011d28, buf=0x7f81040133f8 "\377\212\230\063") at ../../mysql-8.0.18/sql/handler.cc:7740
#17 0x00000000012a7f05 in write_record (thd=thd@entry=0x7f80e4000dd0, table=table@entry=0x7f8104004090, info=info@entry=0x7f81545ad940, update=update@entry=0x0) at ../../mysql-8.0.18/sql/sql_insert.cc:1946
#18 0x00000000012b0e65 in Sql_cmd_load_table::read_sep_field (this=0x7f80e400aab8, thd=0x7f80e4000dd0, info=..., table_list=0x7f80e400abc8, read_info=..., enclosed=..., skip_lines=0) at ../../mysql-8.0.18/sql/sql_load.cc:1043
#19 0x00000000012b467c in Sql_cmd_load_table::execute_inner (this=<optimized out>, thd=0x7f80e4000dd0, handle_duplicates=DUP_ERROR) at ../../mysql-8.0.18/sql/sql_load.cc:542
#20 0x00000000012b4c63 in Sql_cmd_load_table::execute (this=0x7f80e400aab8, thd=0x7f80e4000dd0) at ../../mysql-8.0.18/sql/sql_load.cc:2000
#21 0x0000000000e8cbab in mysql_execute_command (thd=0x7f80e4000dd0, first_level=<optimized out>) at ../../mysql-8.0.18/sql/sql_parse.cc:3450
#22 0x0000000000e8e8b7 in mysql_parse (thd=thd@entry=0x7f80e4000dd0, parser_state=parser_state@entry=0x7f81545af5f0) at ../../mysql-8.0.18/sql/sql_parse.cc:5257
#23 0x0000000000e91244 in dispatch_command (thd=0x7f80e4000dd0, com_data=<optimized out>, command=COM_QUERY) at ../../mysql-8.0.18/sql/sql_parse.cc:1765
#24 0x0000000000e91cb4 in do_command (thd=thd@entry=0x7f80e4000dd0) at ../../mysql-8.0.18/sql/sql_parse.cc:1273
#25 0x0000000000fa4c78 in handle_connection (arg=arg@entry=0x4fc93d0) at ../../mysql-8.0.18/sql/conn_handler/connection_handler_per_thread.cc:302
#26 0x00000000023b04ac in pfs_spawn_thread (arg=0x504cfa0) at ../../../mysql-8.0.18/storage/perfschema/pfs.cc:2854
#27 0x00007f818d7986db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#28 0x00007f818b80771f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

可知其是 lock_move_rec_list_end 从其他页的锁转移过来的, 原始锁为: 0x7f8165f0b740

以 0x7f8165f0b740 为例, 查询日志, 获得: 

```
Thread 35 "mysqld" hit Breakpoint 3, RecLock::lock_add (this=0x7f814e7faa60, lock=0x7f8165f0b740, add_to_hash=true) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1254
1254    ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc: No such file or directory.
$1647 = (ib_lock_t *) 0x7f8165f0b740
#3  0x0000000002006306 in lock_rec_inherit_to_gap (heir_block=heir_block@entry=0x7f8163dc0180, block=block@entry=0x7f8163e1f880, heir_heap_no=2, heap_no=heap_no@entry=1) at ../../../mysql-8.0.18/storage/innobase/include/lock0priv.ic:264
264     ../../../mysql-8.0.18/storage/innobase/include/lock0priv.ic: No such file or directory.
lock = 0x7f8165f0b5c0
#0  RecLock::lock_add (this=0x7f814e7faa60, lock=0x7f8165f0b740, add_to_hash=true) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1254
#1  0x0000000002004559 in RecLock::create (this=this@entry=0x7f814e7faa60, trx=trx@entry=0x7f8178d7cdd0, add_to_hash=add_to_hash@entry=true, prdt=prdt@entry=0x0) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1329
#2  0x0000000002004c9c in lock_rec_add_to_queue (type_mode=<optimized out>, block=0x7f8163dc0180, heap_no=<optimized out>, index=<optimized out>, trx=0x7f8178d7cdd0, we_own_trx_mutex=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:1551
#3  0x0000000002006306 in lock_rec_inherit_to_gap (heir_block=heir_block@entry=0x7f8163dc0180, block=block@entry=0x7f8163e1f880, heir_heap_no=2, heap_no=heap_no@entry=1) at ../../../mysql-8.0.18/storage/innobase/include/lock0priv.ic:264
#4  0x0000000002006524 in lock_update_merge_right (right_block=0x7f8163dc0180, orig_succ=0x7f816600407d "\200\063f\\", left_block=0x7f8163e1f880) at ../../../mysql-8.0.18/storage/innobase/lock/lock0lock.cc:3181
#5  0x00000000021a3fa6 in btr_compress (cursor=cursor@entry=0x7f8174482868, adjust=adjust@entry=0, mtr=mtr@entry=0x7f814e7fb560) at ../../../mysql-8.0.18/storage/innobase/btr/btr0btr.cc:3378
#6  0x00000000021af7f8 in btr_cur_compress_if_useful (cursor=cursor@entry=0x7f8174482868, adjust=adjust@entry=0, mtr=mtr@entry=0x7f814e7fb560) at ../../../mysql-8.0.18/storage/innobase/btr/btr0cur.cc:4559
#7  0x00000000021b0be6 in btr_cur_pessimistic_delete (err=err@entry=0x7f814e7fb234, has_reserved_extents=has_reserved_extents@entry=0, cursor=cursor@entry=0x7f8174482868, flags=flags@entry=0, rollback=rollback@entry=false, trx_id=9056, undo_no=0, rec_type=14, mtr=0x7f814e7fb560) at ../../../mysql-8.0.18/storage/innobase/btr/btr0cur.cc:4852
#8  0x00000000020ddfe3 in row_purge_remove_clust_if_poss_low (node=0x7f81744827b8, mode=65569) at ../../../mysql-8.0.18/storage/innobase/include/btr0pcur.h:719
#9  0x00000000020e0bbd in row_purge_remove_clust_if_poss (node=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:233
#10 row_purge_del_mark (node=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:671
#11 row_purge_record_func (thd=0x7f812c000b20, updated_extern=<optimized out>, undo_rec=0x7f8134324500 "\002rN", node=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:1081
#12 row_purge (thr=<optimized out>, undo_rec=<optimized out>, node=<optimized out>) at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:1155
#13 row_purge_step (thr=thr@entry=0x7f81744825f8) at ../../../mysql-8.0.18/storage/innobase/row/row0purge.cc:1227
#14 0x0000000002086128 in que_thr_step (thr=0x7f81744825f8) at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:923
#15 que_run_threads_low (thr=0x7f81744825f8) at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:974
#16 que_run_threads (thr=thr@entry=0x7f81744825f8) at ../../../mysql-8.0.18/storage/innobase/que/que0que.cc:1009
#17 0x00000000021150dc in srv_task_execute () at ../../../mysql-8.0.18/storage/innobase/srv/srv0srv.cc:2724
#18 srv_worker_thread () at ../../../mysql-8.0.18/storage/innobase/srv/srv0srv.cc:2763
#19 0x000000000202eac5 in std::__invoke_impl<void, void (*&)()> (__f=<synthetic pointer>: <optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:59
#20 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#21 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:400
#22 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:484
#23 Runnable::operator()<void (*)()> (f=@0x7f81745d1158: 0x2114ce0 <srv_worker_thread()>, this=0x7f81745d1160) at ../../../mysql-8.0.18/storage/innobase/include/os0thread-create.h:101
#24 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:60
#25 std::__invoke<Runnable, void (*)()> (__fn=...) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#26 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x7f81745d1158) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:244
#27 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x7f81745d1158) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:253
#28 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x7f81745d1150) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:196
#29 0x00000000025d3f0f in execute_native_thread_routine ()
#30 0x00007f818d7986db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#31 0x00007f818b80771f in clone () from /lib/x86_64-linux-gnu/libc.so.6
[Switching to Thread 0x7f815c0a1700 (LWP 31485)]
``` 

该锁是在表purge的过程中, 由lock_rec_inherit_to_gap创建, 也就是在删除记录时, 将记录所持有的锁, 转移到右侧的记录

lock_rec_inherit_to_gap收到 trx->lock->inherit_all的影响

如果 标记了 trx->lock->inherit_all, 那么认为该事务具有一个用户层面(非语句层面)(或为约束检查创建的锁) 的锁逻辑, 于是需要将锁转换成GAP, 用于保护用户层面的逻辑正确

参考代码说明: 

```
  /** Used to indicate that every lock of this transaction placed on a record
  which is being purged should be inherited to the gap.
  Readers should hold a latch on the lock they'd like to learn about wether or
  not it should be inherited.
  Writers who want to set it to true, should hold a latch on the lock-sys queue
  they intend to add a lock to.
  Writers may set it to false at any time. */
  std::atomic<bool> inherit_all;

```
```
    /* If session is using READ COMMITTED or READ UNCOMMITTED isolation
  level, we do not want locks set by an UPDATE or a DELETE to be
  inherited as gap type locks.  But we DO want S-locks/X-locks(taken for
  replace) set by a consistency constraint to be inherited also then. */

  /* We also dont inherit these locks as gap type locks for DD tables
  because the serialization is guaranteed by MDL on DD tables. */

  /* Constraint checks place LOCK_S or (in case of INSERT ... ON DUPLICATE
  UPDATE... or REPLACE INTO..) LOCK_X on records.
  If such a record is delete-marked, it may then become purged, and
  lock_rec_inheirt_to_gap will be called to decide the fate of each lock on it:
  either it will be inherited as gap lock, or discarded.
  In READ COMMITTED and less restricitve isolation levels we generaly avoid gap
  locks, but we make an exception for precisely this situation: we want to
  inherit locks created for constraint checks.
  More precisely we need to keep inheriting them only for the duration of the
  query which has requested them, as such inserts have two phases : first they
  check for constraints, then they do actuall row insert, and they trust that
  the locks set in the first phase will survive till the second phase.
  It is not easy to tell if a particular lock was created for constraint check
  or not, because we do not store this bit of information on it.
  What we do, is we use a heuristic: whenever a trx requests a lock with
  lock_duration_t::AT_LEAST_STATEMENT we set trx->lock.inherit_all, meaning that
  locks of this trx need to be inherited.
  And we clear trx->lock.inherit_all on statement end. */
``` 

此处MySQL做错的逻辑: It is not easy to tell if a particular lock was created for constraint check or not, 所以粗暴地将不合适的事务标记为 inherit_all

# 结论

整体逻辑: 

  1. 机制1: MySQL的设计中, 为处理好 delete与并发insert同时进行的场景, 在被删除的记录上的锁, 会被转换成GAP锁 (即使隔离级别是RC或者RUC, 也需要转换成GAP锁), 其必要性在之后说明
  2. 故障现象中: 
     1. 用户先进行了delete, delete成功, 但实际记录的purge是后台异步进行
     2. 然后用户进行了 load data, 相当于insert. 同时 purge继续进行, 旧记录被purge时, 将锁转移给其右侧记录, 根据机制1, 锁转换成GAP
     3. 随着引擎树的扩容调整, GAP有机会被转移到supremum记录上
     4. 当事务A和B同时 进行insert操作, 按照上述步骤持有了某数据页的GAP, 要向对方持有的数据页插入数据, 就会发生死锁. 
     5. 死锁现象是两个事务 都持有 某页的S (supremum), 等待 对方持有页的 X 锁

机制1的必要性: 

  - 参考 <http://mysql.taobao.org/monthly/2015/06/02/> 的问题1
    - 插入记录的操作流程
      - 简单查重: 上S,GAP锁
      - 插入: 
        - 快速排他: 上X,GAP,INTENTION
        - 正式插入: 行锁X
    - delete (A线程) 与 并发 insert (B/C线程) 同时进行, 数据表中有唯一键
    - A线程 删除的记录, 尚未进行purge
    - B/C线程 插入相同的记录, 分别于A线程删除的记录, 进行简单查重. 
      - 如果简单查重的锁 为 S,REC, 那么B/C可以同时通过查重阶段, 并且进入插入阶段后, 串行插入. 这样会破坏唯一键的语义
      - 所以: 简单查重阶段需要上 S,GAP 锁, 这样通过简单查重阶段后, B/C的快速排他阶段, 都会等待对方的S,GAP锁, 形成死锁.
      - 这把GAP锁, 主要为了与快速排他阶段的GAP锁互斥, 保证 快速排他阶段, 仍保持查重的语义
  - 与之相关的MySQL测试用例, 参看: <https://github.com/mysql/mysql-server/commit/58d2400343656cf82ff5fbb6680beaf46f5de974>

逻辑整理: 

  1. delete后, 行清理 (purge) 是异步的
  2. purge时, 需要对行上已有的GAP锁进行转移. (转移的必要性需要举例说明)
  3. insert, 为了提高两方面性能 (查重性能 和 在区间内并发插入的能力), 分成了三个阶段 (查重, 排他, 插入)
  4. 根据 [http://mysql.taobao.org/monthly/2015/06/02](<http://mysql.taobao.org/monthly/2015/06/02/>)的解释, Insert进入查重阶段, 需要上S,GAP锁
     1. 这把S,GAP锁, 不是为了维护隔离级别, 而是为了让insert的前两阶段互斥, 避免unique键的唯一语义丢失
  5. 有了GAP锁, 才会造成问题中的死锁

解决方案: 

  1. 改造脚本, 在delete后, 等待 show engine innodb status\G 中显示 purge线程idle, 或者在delete后等待一段时间, 在进行load data操作

# 其他知识

  1. 停止purge的方法: 
     1. flush tables tbname for export
     2. debug变量: innodb_purge_stop_now  

  2. 一问一实验
     1. 通过gdb诊断 锁来源
     2. 根据commit, 查找测试用例
