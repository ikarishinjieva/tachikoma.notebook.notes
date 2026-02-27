---
title: 20220419 - 国债故障
confluence_page_id: 1737095
created_at: 2022-04-22T15:16:38+00:00
updated_at: 2022-04-22T15:24:15+00:00
---

<https://support.actionsky.com/service_desk/browse/BEIJ-2452>

# 原因分析

![image2022-4-22 23:14:36.png](/assets/01KJBYJ6VFVAX9WQ0HD1G3TPEM/image2022-4-22%2023%3A14%3A36.png)

buffer pool中, 关于SYS_TABLES系统表的数据页, 有125w页被设置成了锁定页, 不能从buffer pool中刷盘, 只能保存在刷盘队列中

(正常情况下, SYS_TABLES 应被刷盘到 innodb_temporary 表空间)

目前不知道为什么SYS_TABLES设置为锁定页后, 没有正常解除, 需要花费时间梳理MySQL的相关逻辑, 找到复现场景.

\------

SYS_TABLES的锁定页充满了刷盘队列尾端, 导致MySQL刷盘时, 需要从尾端注意扫描, 直至第一个非锁定页. 

所以导致刷盘缓慢, 相关逻辑 (buf_do_flush_list_batch): 

![image2022-4-22 23:14:55.png](/assets/01KJBYJ6VFVAX9WQ0HD1G3TPEM/image2022-4-22%2023%3A14%3A55.png)

从故障时的堆栈采样中可证实

\------

刷盘慢, 会导致 dict_stats_save 线程, 在向数据字典保存统计数据时, 在持有锁的情况下, 等待刷盘逻辑: 

等待堆栈可以验证: 

```
Thread 1065 (Thread 0x7fb71affd700 (LWP 22000)):
#0  0x00007fd68fe62945 in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
#1  0x000000000128e85b in wait (this=0x1c510df8) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/os/os0event.cc:179
#2  os_event::wait_low (this=0x1c510df8, reset_sig_count=2158606) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/os/os0event.cc:366
#3  0x0000000001335cb9 in sync_array_wait_event (arr=0x2ce8448, cell=@0x7fb71affb9c8: 0x7fd683115268) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/sync/sync0arr.cc:483
#4  0x000000000121e524 in TTASEventMutex<GenericPolicy>::wait (this=0x3005140, filename=0x181d000 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0flu.cc", line=2056, spin=4) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ut0mutex.ic:97
#5  0x000000000121e69b in spin_and_try_lock (line=2056, filename=0x181d000 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0flu.cc", this=0x3005140, max_delay=6, max_spins=60) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ib0mutex.h:858
#6  enter (line=2056, filename=0x181d000 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0flu.cc", max_delay=6, max_spins=30, this=0x3005140) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ib0mutex.h:715
#7  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x3005140, n_spins=30, n_delay=6, name=0x181d000 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0flu.cc", line=2056) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ib0mutex.h:995
#8  0x00000000013ca96c in buf_flush_wait_flushed (new_oldest=8543560656381) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/buf/buf0flu.cc:2056
#9  0x0000000001267bf9 in log_preflush_pool_modified_pages (new_oldest=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/log/log0log.cc:1539
#10 0x000000000126b0d9 in log_checkpoint_margin () at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/log/log0log.cc:1981
#11 log_check_margins () at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/log/log0log.cc:2073
#12 0x00000000012aefcd in log_free_check () at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/log0log.ic:499
#13 que_run_threads_low (thr=0x7fb6fc007d80) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/que/que0que.cc:1113
#14 que_run_threads (thr=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/que/que0que.cc:1159
#15 0x00000000012af8ee in que_eval_sql (info=0x7fb6fc002c68, sql=0x1822120 "PROCEDURE TABLE_STATS_SAVE () IS\nBEGIN\nDELETE FROM \"mysql/innodb_table_stats\"\nWHERE\ndatabase_name = :database_name AND\ntable_name = :table_name;\nINSERT INTO \"mysql/innodb_table_stats\"\nVALUES\n(\n:databa"..., reserve_dict_mutex=0, trx=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/que/que0que.cc:1236
#16 0x000000000140bfcc in dict_stats_exec_sql (pinfo=0x7fb6fc002c68, sql=0x1822120 "PROCEDURE TABLE_STATS_SAVE () IS\nBEGIN\nDELETE FROM \"mysql/innodb_table_stats\"\nWHERE\ndatabase_name = :database_name AND\ntable_name = :table_name;\nINSERT INTO \"mysql/innodb_table_stats\"\nVALUES\n(\n:databa"..., trx=0x7fd682d1c070) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/dict/dict0stats.cc:319
#17 0x000000000140e776 in dict_stats_save (table_orig=<optimized out>, only_for_index=0x0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/dict/dict0stats.cc:2424
#18 0x000000000140fe0a in dict_stats_update (table=0x7fb6e984ad38, stats_upd_option=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/dict/dict0stats.cc:3122
#19 0x000000000141206a in dict_stats_process_entry_from_recalc_pool () at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/dict/dict0stats_bg.cc:363
#20 dict_stats_thread (arg=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/dict/dict0stats_bg.cc:452
#21 0x00007fd68fe5edd5 in start_thread () from /lib64/libpthread.so.0
#22 0x00007fd68e918b3d in clone () from /lib64/libc.so.6
``` 

dict_stats_save等待刷盘时, 持有的锁是: dict_operation_lock 和 dict_sys: 

![image2022-4-22 23:15:17.png](/assets/01KJBYJ6VFVAX9WQ0HD1G3TPEM/image2022-4-22%2023%3A15%3A17.png)

这两把锁, 会阻塞其他操作, 比如 下SQL时候的使用内部临时表的操作 (影响performance_schema的监控SQL, 也会影响正常查询): 

```
Thread 1059 (Thread 0x7fb7301b2700 (LWP 22040)):
#0  0x00007fd68fe62945 in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
#1  0x000000000128e85b in wait (this=0xcd8949d8) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/os/os0event.cc:179
#2  os_event::wait_low (this=0xcd8949d8, reset_sig_count=53999) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/os/os0event.cc:366
#3  0x0000000001335cb9 in sync_array_wait_event (arr=0x2ce8448, cell=@0x7fb7301af5c8: 0x7fd683115168) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/sync/sync0arr.cc:483
#4  0x000000000121e524 in TTASEventMutex<GenericPolicy>::wait (this=0xcd8948e8, filename=0x17f4fa0 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc", line=5623, spin=4) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ut0mutex.ic:97
#5  0x000000000121e69b in spin_and_try_lock (line=5623, filename=0x17f4fa0 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc", this=0xcd8948e8, max_delay=6, max_spins=60) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ib0mutex.h:858
#6  enter (line=5623, filename=0x17f4fa0 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc", max_delay=6, max_spins=30, this=0xcd8948e8) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ib0mutex.h:715
#7  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0xcd8948e8, n_spins=30, n_delay=6, name=0x17f4fa0 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc", line=5623) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/include/ib0mutex.h:995
#8  0x00000000012123fe in innobase_build_index_translation (share=0x7fb6ec943e40, ib_table=0x7fb6ec015048, table=0x7fb6ec01ade0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:5623
#9  ha_innobase::open (this=0x7fb6ec015d30, name=0x7fb6ec01c038 "/mysqlbase/data/mysql/tmp/13306/#sql_553e_0", mode=<optimized out>, test_if_locked=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/innobase/handler/ha_innodb.cc:6069
#10 0x000000000085086e in handler::ha_open (this=0x7fb6ec015d30, table_arg=<optimized out>, name=0x7fb6ec01c038 "/mysqlbase/data/mysql/tmp/13306/#sql_553e_0", mode=2, test_if_locked=516) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/handler.cc:2774
#11 0x0000000000dc511d in open_tmp_table (table=0x7fb6ec01ade0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_tmp_table.cc:2072
#12 0x0000000000dc663f in instantiate_tmp_table (table=0x7fb6ec01ade0, keyinfo=<optimized out>, start_recinfo=<optimized out>, recinfo=<optimized out>, options=<optimized out>, big_tables=<optimized out>, trace=0x7fb6ec01e668) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_tmp_table.cc:2389
#13 0x0000000000dc95db in create_tmp_table (thd=0x7fb6ec01c510, param=0x7fb6ec0192e0, fields=..., group=0x0, distinct=false, save_sum_fields=false, select_options=4096, rows_limit=18446744073709551615, table_alias=0x7fb6ec012a50 "processlist") at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_tmp_table.cc:1528
#14 0x0000000000da0e58 in create_schema_table (thd=0x7fb6ec01c510, table_list=0x7fb6ec012a60) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:7773
#15 0x0000000000d935c7 in mysql_schema_table (thd=0x7fb6ec01c510, lex=0x7fb6ec01e680, table_list=0x7fb6ec012a60) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_show.cc:8002
#16 0x0000000000cfa9ac in open_and_process_table (ot_ctx=0x7fb7301b04a0, has_prelocking_list=false, prelocking_strategy=0x7fb7301b0570, flags=0, counter=0x7fb6ec01e740, tables=0x7fb6ec012a60, lex=0x7fb6ec01e680, thd=0x7fb6ec01c510) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:5116
#17 open_tables (thd=0x7fb6ec01c510, start=0x7fb7301b0558, counter=0x7fb6ec01e740, flags=0, prelocking_strategy=0x7fb7301b0570) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:5851
#18 0x0000000000cfb9a2 in open_tables_for_query (thd=0x7fb6ec01c510, tables=0x7fb6ec012a60, flags=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:6626
#19 0x0000000000d4af06 in execute_sqlcom_select (thd=0x7fb6ec01c510, all_tables=0x7fb6ec012a60) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:5126
#20 0x0000000000d4ecfa in mysql_execute_command (thd=0x7fb6ec01c510, first_level=true) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:2826
#21 0x0000000000d5085d in mysql_parse (thd=0x7fb6ec01c510, parser_state=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:5584
#22 0x0000000000d51b05 in dispatch_command (thd=0x7fb6ec01c510, com_data=0x7fb7301b1da0, command=COM_QUERY) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:1491
#23 0x0000000000d529e4 in do_command (thd=0x7fb6ec01c510) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_parse.cc:1032
#24 0x0000000000e24dcc in handle_connection (arg=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/conn_handler/connection_handler_per_thread.cc:313
#25 0x0000000001188c74 in pfs_spawn_thread (arg=0xcdaa2660) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/perfschema/pfs.cc:2197
#26 0x00007fd68fe5edd5 in start_thread () from /lib64/libpthread.so.0
#27 0x00007fd68e918b3d in clone () from /lib64/libc.so.6
``` 

\------

探活SQL超时, 会导致DMP判断MySQL故障

刷盘操作会在检索完锁定页后继续进行, 所以刷盘操作会阻塞一阵 (在业务压力下, 接近10min), 然后正常一批, 到下一轮刷盘继续阻塞一阵

DMP也就会判断节点不断故障, 又不断恢复

\------

复制回放的MTS会等待commit order, 即后续事务会等待前序事务的SQL执行完成后才会继续, 而前序事务阻塞, 所以会看到processlist中的MTS等待: 

![image2022-4-22 23:15:34.png](/assets/01KJBYJ6VFVAX9WQ0HD1G3TPEM/image2022-4-22%2023%3A15%3A34.png)

# 为什么会有半同步断开

```
2022-04-19T16:44:30.332399+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect

2022-04-19T16:53:33.334536+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect

2022-04-19T17:02:23.165504+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T17:11:06.207609+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T17:23:51.327325+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T17:33:00.977683+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T17:48:13.413747+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T17:51:16.911853+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect

2022-04-19T17:58:18.350097+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect

2022-04-19T18:08:45.781601+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T18:16:55.844436+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T18:27:24.242718+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
(long semaphore 断在此处)

2022-04-19T18:31:24.191315+08:00 2980583 [ERROR] Semi-sync slave net_flush() reply failed
Slave I/O thread reconnect
``` 

dict_stats_save, 在持有DD锁时, 等待刷脏. 

而Slave I/O 线程, 会在接受relay log后, 会从DD中查询表结构来检查表的合法性, 所以slave IO线程会在接收到event后阻塞: 

```
Thread 19 (Thread 0x7fb67920d700 (LWP 109717)):
#0  0x00007fd68fe654cd in __lll_lock_wait () from /lib64/libpthread.so.0
#1  0x00007fd68fe60e01 in _L_lock_1022 () from /lib64/libpthread.so.0
#2  0x00007fd68fe60da2 in pthread_mutex_lock () from /lib64/libpthread.so.0
#3  0x0000000000cf65e3 in native_mutex_lock (mutex=0x20baf60 <LOCK_open>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/include/thr_mutex.h:91
#4  my_mutex_lock (mp=0x20baf60 <LOCK_open>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/include/thr_mutex.h:189
#5  inline_mysql_mutex_lock (src_line=2686, src_file=0x15a1bc8 "/export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc", that=0x20baf60 <LOCK_open>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/include/mysql/psi/mysql_thread.h:722
#6  check_if_table_exists (thd=0x7faeb000c6e0, table=0x7fb67920c2d0, exists=0x7fb67920c09f) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:2686
#7  0x0000000000cf9750 in open_table (thd=0x7faeb000c6e0, table_list=0x7fb67920c2d0, ot_ctx=0x7fb67920c160) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:3283
#8  0x0000000000cfb516 in open_and_process_table (ot_ctx=0x7fb67920c160, has_prelocking_list=false, prelocking_strategy=0x7fb67920c920, flags=10251, counter=0x7fb67920c23c, tables=0x7fb67920c2d0, lex=0x7faeb000e850, thd=0x7faeb000c6e0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:5226
#9  open_tables (thd=0x7faeb000c6e0, start=0x7fb67920c218, counter=0x7fb67920c23c, flags=10251, prelocking_strategy=0x7fb67920c920) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:5851
#10 0x0000000000cfba2e in open_and_lock_tables (thd=<optimized out>, tables=0x7fb67920c2d0, flags=10251, prelocking_strategy=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:6571
#11 0x0000000000cfbad2 in open_n_lock_single_table (thd=<optimized out>, table_l=0x7fb67920c2d0, lock_type=<optimized out>, flags=<optimized out>, prelocking_strategy=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.cc:6409
#12 0x0000000000f3717f in open_n_lock_single_table (flags=<optimized out>, lock_type=<optimized out>, table_l=0x7fb67920c2d0, thd=0x7faeb000c6e0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/sql_base.h:477
#13 System_table_access::open_table (this=0x7facb001bd50, thd=0x7faeb000c6e0, dbstr=..., tbstr=..., max_num_field=25, lock_type=TL_WRITE, table=0x7fb67920ca28, backup=0x7fb67920c9a0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/rpl_table_access.cc:63
#14 0x0000000000f6e447 in Rpl_info_table::do_flush_info (this=0x7facb04cf6b0, force=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/rpl_info_table.cc:185
#15 0x0000000000f56df9 in flush_info (force=false, this=<optimized out>) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/rpl_info_handler.h:101
#16 Master_info::flush_info (this=0x7facb0030da0, force=false) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/rpl_mi.cc:281
#17 0x0000000000f4a9ef in flush_master_info (mi=<optimized out>, force=false) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/rpl_slave.cc:1459
#18 0x0000000000f4ef28 in handle_slave_io (arg=0x7facb0030da0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/sql/rpl_slave.cc:5885
#19 0x0000000001188c74 in pfs_spawn_thread (arg=0x7fb5c800d6d0) at /export/home/pb2/build/sb_0-37309218-1576676677.02/mysql-5.7.29/storage/perfschema/pfs.cc:2197
#20 0x00007fd68fe5edd5 in start_thread () from /lib64/libpthread.so.0
#21 0x00007fd68e918b3d in clone () from /lib64/libc.so.6
``` 

阻塞过程中, slave I/O的网络连接上不再收发数据包.

直到本轮刷脏完成, dict_stats_save的锁解开, slave I/O线程才可以继续收发数据包. 

但由于网络连接空闲时间过长, 会被网络中继设备认为连接超时而清理. 所以再次收发数据包时, 会报网络连接错误.

错误同时体现在收发环节, 因此在慢日志中: 

  1. 半同步和复制的连接报错会同时出现
  2. 每当出现 半同步和复制的连接报错, long semaphore都会消失. 然后再次出现 (因为下一轮刷盘仍会产生同样的阻塞)

# 运维建议

  1. 可以升级MySQL 至 5.7.35, MySQL对刷盘队列的扫描逻辑进行了优化: <https://github.com/mysql/mysql-server/commit/d590d77180d79c17cc36f69dd6a1892941149ab7>
  2. 增加对buffer pool刷盘队列: flush_list的监控; 增加 buffer pool page (SYS_TABLES)的监控.
  3. 对于故障节点, 重启可清空刷盘队列
  4. 需要时间来整理MySQL的锁定页逻辑, 目前没有已知bug与之相关, 需要摸索触发条件

# 知识点

  1. innodb_buffer_pool 表, 可以看到buffer pool的内容, 其中: 
     1. 当 page 进入 flush_list 时, 会设置 old_modification, 当刷盘成功后, 会从flush_list中删除, 并清空old_modification
     2. page->fix_count, 表示MySQL逻辑中, 需要page固定在buffer pool中不被请出 (IO写入期间 / 压缩过程中 / bulk操作)
     3. IO_FIX, 进行IO操作前, 会设置为相应的IO类型, IO操作结束后, 设置为 BUF_IO_NONE
  2. dict_stats_save 在表增减, 以及表数据变更10%时, 会触发
