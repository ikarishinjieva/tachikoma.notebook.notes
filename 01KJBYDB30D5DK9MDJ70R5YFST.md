---
title: 20210513 - 百胜cpu高的探索
confluence_page_id: 754119
created_at: 2021-05-13T14:12:45+00:00
updated_at: 2021-05-18T10:17:04+00:00
---

# 现象

CPU高, 没有特定SQL出问题

perf截图: 

![image2021-5-17 11:39:17.png](/assets/01KJBYDB30D5DK9MDJ70R5YFST/image2021-5-17%2011%3A39%3A17.png)

![image2021-5-17 11:39:28.png](/assets/01KJBYDB30D5DK9MDJ70R5YFST/image2021-5-17%2011%3A39%3A28.png)

# 分析

探索PolicyMutex >::enter的入口: 执行update操作时, 一共四类堆栈: 

  - 申请可用的 undo segment, 所在segment上

```
#0  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x4fd6e00, n_spins=30, n_delay=6,
    name=0x232c5a0 "/export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/trx/trx0trx.cc", line=1184)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/include/ib0mutex.h:982
#1  0x0000000001b500f4 in get_next_redo_rseg (max_undo_logs=128, n_tablespaces=0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/trx/trx0trx.cc:1184
#2  0x0000000001b504f7 in trx_assign_rseg_low (max_undo_logs=128, n_tablespaces=0,
    rseg_type=TRX_RSEG_TYPE_REDO)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/trx/trx0trx.cc:1277
#3  0x0000000001b54d94 in trx_set_rw_mode (trx=0x7f9beb3018c0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/trx/trx0trx.cc:3220
#4  0x00000000019b55db in lock_table (flags=0, table=0x7f9b98014900, mode=LOCK_IX,
    thr=0x7f9b84014f58)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/lock/lock0lock.cc:4035
#5  0x0000000001ac05ea in row_search_mvcc (buf=0x7f9b84010bd0 "\377", mode=PAGE_CUR_GE,
    prebuilt=0x7f9b84014820, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0sel.cc:5048
#6  0x000000000192d86e in ha_innobase::index_read (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key_ptr=0x7f9b84018a00 "\n", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:8740
#7  0x0000000000f69b9e in handler::index_read_map (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.h:2808
#8  0x0000000000f5b240 in handler::ha_index_read_map (this=0x7f9b840108e0,
    buf=0x7f9b84010bd0 "\377", key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:3031
#9  0x0000000000f65320 in handler::read_range_first (this=0x7f9b840108e0, start_key=0x7f9b840109c8,
    end_key=0x7f9b840109e8, eq_range_arg=true, sorted=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:7369
#10 0x0000000000f6327f in handler::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6439
#11 0x0000000000f64169 in DsMrr_impl::dsmrr_next (this=0x7f9b84010b40, range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6823
#12 0x000000000193fd54 in ha_innobase::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:20582
#13 0x0000000001788f18 in QUICK_RANGE_SELECT::get_next (this=0x7f9b84011e20)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/opt_range.cc:11233
#14 0x00000000014b5f64 in rr_quick (info=0x7f9bd8171a30)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/records.cc:398
#15 0x000000000164e080 in mysql_update (thd=0x7f9b84000dd0, fields=..., values=...,
    limit=18446744073709551615, handle_duplicates=DUP_ERROR, found_return=0x7f9bd8171d88,
    updated_return=0x7f9bd8171d80)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:812
#16 0x0000000001654688 in Sql_cmd_update::try_single_table_update (this=0x7f9b84005818,
    thd=0x7f9b84000dd0, switch_to_multitable=0x7f9bd8171e2f)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:2891
#17 0x0000000001654bd5 in Sql_cmd_update::execute (this=0x7f9b84005818, thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:3018
#18 0x0000000001597c33 in mysql_execute_command (thd=0x7f9b84000dd0, first_level=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:3606
#19 0x000000000159db38 in mysql_parse (thd=0x7f9b84000dd0, parser_state=0x7f9bd8173670)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:5570
#20 0x0000000001592714 in dispatch_command (thd=0x7f9b84000dd0, com_data=0x7f9bd8173de0,
    command=COM_QUERY)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1484
#21 0x0000000001591559 in do_command (thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1025
#22 0x00000000016c7f75 in handle_connection (arg=0x4deddb0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/conn_handler/connection_handler_per_thread.cc:306
#23 0x0000000001d72280 in pfs_spawn_thread (arg=0x4daa010)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/perfschema/pfs.cc:2190
#24 0x00007f9bf99ec6db in start_thread (arg=0x7f9bd8174700) at pthread_create.c:463
#25 0x00007f9bf838571f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

  - 开启读写事务, 所在事务系统trx_sys上, 用于修改读写事务列表

```
(gdb) bt
#0  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x4cf9e78, n_spins=30, n_delay=6,
    name=0x232c5a0 "/export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/trx/trx0trx.cc", line=3224)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/include/ib0mutex.h:982
#1  0x0000000001b54e09 in trx_set_rw_mode (trx=0x7f9beb3018c0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/trx/trx0trx.cc:3224
#2  0x00000000019b55db in lock_table (flags=0, table=0x7f9b98014900, mode=LOCK_IX,
    thr=0x7f9b84014f58)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/lock/lock0lock.cc:4035
#3  0x0000000001ac05ea in row_search_mvcc (buf=0x7f9b84010bd0 "\377", mode=PAGE_CUR_GE,
    prebuilt=0x7f9b84014820, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0sel.cc:5048
#4  0x000000000192d86e in ha_innobase::index_read (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key_ptr=0x7f9b84018a00 "\n", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:8740
#5  0x0000000000f69b9e in handler::index_read_map (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.h:2808
#6  0x0000000000f5b240 in handler::ha_index_read_map (this=0x7f9b840108e0,
    buf=0x7f9b84010bd0 "\377", key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:3031
#7  0x0000000000f65320 in handler::read_range_first (this=0x7f9b840108e0, start_key=0x7f9b840109c8,
    end_key=0x7f9b840109e8, eq_range_arg=true, sorted=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:7369
#8  0x0000000000f6327f in handler::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6439
#9  0x0000000000f64169 in DsMrr_impl::dsmrr_next (this=0x7f9b84010b40, range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6823
#10 0x000000000193fd54 in ha_innobase::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:20582
#11 0x0000000001788f18 in QUICK_RANGE_SELECT::get_next (this=0x7f9b84011e20)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/opt_range.cc:11233
#12 0x00000000014b5f64 in rr_quick (info=0x7f9bd8171a30)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/records.cc:398
#13 0x000000000164e080 in mysql_update (thd=0x7f9b84000dd0, fields=..., values=...,
    limit=18446744073709551615, handle_duplicates=DUP_ERROR, found_return=0x7f9bd8171d88,
    updated_return=0x7f9bd8171d80)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:812
#14 0x0000000001654688 in Sql_cmd_update::try_single_table_update (this=0x7f9b84005818,
    thd=0x7f9b84000dd0, switch_to_multitable=0x7f9bd8171e2f)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:2891
#15 0x0000000001654bd5 in Sql_cmd_update::execute (this=0x7f9b84005818, thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:3018
#16 0x0000000001597c33 in mysql_execute_command (thd=0x7f9b84000dd0, first_level=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:3606
#17 0x000000000159db38 in mysql_parse (thd=0x7f9b84000dd0, parser_state=0x7f9bd8173670)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:5570
#18 0x0000000001592714 in dispatch_command (thd=0x7f9b84000dd0, com_data=0x7f9bd8173de0,
    command=COM_QUERY)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1484
#19 0x0000000001591559 in do_command (thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1025
#20 0x00000000016c7f75 in handle_connection (arg=0x4deddb0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/conn_handler/connection_handler_per_thread.cc:306
#21 0x0000000001d72280 in pfs_spawn_thread (arg=0x4daa010)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/perfschema/pfs.cc:2190
#22 0x00007f9bf99ec6db in start_thread (arg=0x7f9bd8174700) at pthread_create.c:463
#23 0x00007f9bf838571f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

  - 锁表, 对锁系统lock_sys上锁, 用于检查锁的兼容性

```
#0  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x4cd8738, n_spins=30, n_delay=6,
    name=0x22ced08 "/export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/lock/lock0lock.cc", line=4038)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/include/ib0mutex.h:982
#1  0x00000000019b5616 in lock_table (flags=0, table=0x7f9b98014900, mode=LOCK_IX,
    thr=0x7f9b84014f58)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/lock/lock0lock.cc:4038
#2  0x0000000001ac05ea in row_search_mvcc (buf=0x7f9b84010bd0 "\377", mode=PAGE_CUR_GE,
    prebuilt=0x7f9b84014820, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0sel.cc:5048
#3  0x000000000192d86e in ha_innobase::index_read (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key_ptr=0x7f9b84018a00 "\n", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:8740
#4  0x0000000000f69b9e in handler::index_read_map (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.h:2808
#5  0x0000000000f5b240 in handler::ha_index_read_map (this=0x7f9b840108e0,
    buf=0x7f9b84010bd0 "\377", key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:3031
#6  0x0000000000f65320 in handler::read_range_first (this=0x7f9b840108e0, start_key=0x7f9b840109c8,
    end_key=0x7f9b840109e8, eq_range_arg=true, sorted=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:7369
#7  0x0000000000f6327f in handler::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6439
#8  0x0000000000f64169 in DsMrr_impl::dsmrr_next (this=0x7f9b84010b40, range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6823
#9  0x000000000193fd54 in ha_innobase::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:20582
#10 0x0000000001788f18 in QUICK_RANGE_SELECT::get_next (this=0x7f9b84011e20)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/opt_range.cc:11233
#11 0x00000000014b5f64 in rr_quick (info=0x7f9bd8171a30)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/records.cc:398
#12 0x000000000164e080 in mysql_update (thd=0x7f9b84000dd0, fields=..., values=...,
    limit=18446744073709551615, handle_duplicates=DUP_ERROR, found_return=0x7f9bd8171d88,
    updated_return=0x7f9bd8171d80)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:812
#13 0x0000000001654688 in Sql_cmd_update::try_single_table_update (this=0x7f9b84005818,
    thd=0x7f9b84000dd0, switch_to_multitable=0x7f9bd8171e2f)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:2891
#14 0x0000000001654bd5 in Sql_cmd_update::execute (this=0x7f9b84005818, thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:3018
#15 0x0000000001597c33 in mysql_execute_command (thd=0x7f9b84000dd0, first_level=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:3606
#16 0x000000000159db38 in mysql_parse (thd=0x7f9b84000dd0, parser_state=0x7f9bd8173670)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:5570
#17 0x0000000001592714 in dispatch_command (thd=0x7f9b84000dd0, com_data=0x7f9bd8173de0,
    command=COM_QUERY)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1484
#18 0x0000000001591559 in do_command (thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1025
#19 0x00000000016c7f75 in handle_connection (arg=0x4deddb0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/conn_handler/connection_handler_per_thread.cc:306
#20 0x0000000001d72280 in pfs_spawn_thread (arg=0x4daa010)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/perfschema/pfs.cc:2190
#21 0x00007f9bf99ec6db in start_thread (arg=0x7f9bd8174700) at pthread_create.c:463
#22 0x00007f9bf838571f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

  - 锁表, 对事务系统trx_sys上锁, 用于检查锁的兼容性

```
(gdb) bt
#0  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x7f9beb3018c0, n_spins=30, n_delay=6,
    name=0x22ced08 "/export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/lock/lock0lock.cc", line=4046)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/include/ib0mutex.h:982
#1  0x00000000019b5665 in lock_table (flags=0, table=0x7f9b98014900, mode=LOCK_IX,
    thr=0x7f9b84014f58)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/lock/lock0lock.cc:4046
#2  0x0000000001ac05ea in row_search_mvcc (buf=0x7f9b84010bd0 "\377", mode=PAGE_CUR_GE,
    prebuilt=0x7f9b84014820, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0sel.cc:5048
#3  0x000000000192d86e in ha_innobase::index_read (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key_ptr=0x7f9b84018a00 "\n", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:8740
#4  0x0000000000f69b9e in handler::index_read_map (this=0x7f9b840108e0, buf=0x7f9b84010bd0 "\377",
    key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.h:2808
#5  0x0000000000f5b240 in handler::ha_index_read_map (this=0x7f9b840108e0,
    buf=0x7f9b84010bd0 "\377", key=0x7f9b84018a00 "\n", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:3031
#6  0x0000000000f65320 in handler::read_range_first (this=0x7f9b840108e0, start_key=0x7f9b840109c8,
    end_key=0x7f9b840109e8, eq_range_arg=true, sorted=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:7369
#7  0x0000000000f6327f in handler::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6439
#8  0x0000000000f64169 in DsMrr_impl::dsmrr_next (this=0x7f9b84010b40, range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:6823
#9  0x000000000193fd54 in ha_innobase::multi_range_read_next (this=0x7f9b840108e0,
    range_info=0x7f9bd8171520)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:20582
#10 0x0000000001788f18 in QUICK_RANGE_SELECT::get_next (this=0x7f9b84011e20)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/opt_range.cc:11233
#11 0x00000000014b5f64 in rr_quick (info=0x7f9bd8171a30)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/records.cc:398
#12 0x000000000164e080 in mysql_update (thd=0x7f9b84000dd0, fields=..., values=...,
    limit=18446744073709551615, handle_duplicates=DUP_ERROR, found_return=0x7f9bd8171d88,
    updated_return=0x7f9bd8171d80)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:812
#13 0x0000000001654688 in Sql_cmd_update::try_single_table_update (this=0x7f9b84005818,
    thd=0x7f9b84000dd0, switch_to_multitable=0x7f9bd8171e2f)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:2891
#14 0x0000000001654bd5 in Sql_cmd_update::execute (this=0x7f9b84005818, thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_update.cc:3018
#15 0x0000000001597c33 in mysql_execute_command (thd=0x7f9b84000dd0, first_level=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:3606
#16 0x000000000159db38 in mysql_parse (thd=0x7f9b84000dd0, parser_state=0x7f9bd8173670)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:5570
#17 0x0000000001592714 in dispatch_command (thd=0x7f9b84000dd0, com_data=0x7f9bd8173de0,
    command=COM_QUERY)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1484
#18 0x0000000001591559 in do_command (thd=0x7f9b84000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1025
#19 0x00000000016c7f75 in handle_connection (arg=0x4deddb0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/conn_handler/connection_handler_per_thread.cc:306
#20 0x0000000001d72280 in pfs_spawn_thread (arg=0x4daa010)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/perfschema/pfs.cc:2190
#21 0x00007f9bf99ec6db in start_thread (arg=0x7f9bd8174700) at pthread_create.c:463
#22 0x00007f9bf838571f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

TTASEventMutex 仅有is_free中调用了ut_delay

# 尝试1 - 调整innodb_sync_spin_loops

调整innodb_sync_spin_loops=30000

使用并发压力: 

```
~/opt/mysql/5.7.25/bin/mysqlslap --create-schema='test' --query='update test.test set a=rand() where a=rand()' --host=127.0.0.1 --port=5725 --user=msandbox --password=msandbox --no-drop --iterations=1 --concurrency=100 --number-of-queries=100000
``` 

可复现类似的perf: 

![image2021-5-17 16:45:40.png](/assets/01KJBYDB30D5DK9MDJ70R5YFST/image2021-5-17%2016%3A45%3A40.png)

# 尝试2 - 调整innodb_sync_spin_loops

调整 innodb_sync_spin_loops = 1000

可复现类似的perf: 

![image2021-5-17 16:53:14.png](/assets/01KJBYDB30D5DK9MDJ70R5YFST/image2021-5-17%2016%3A53%3A14.png)

# CPU微架构的影响

  - 参考: <https://tech.meituan.com/2020/04/16/intel-pause-mysql.html>
  - 其中有测试PAUSE的CPU周期的程序, 经测试skylake框架CPU为100周期, 其他框架为10周期

百胜的CPU为 Intel(R) Xeon(R) Gold 5117 CPU @ 2.00GHz, 属于skylake架构, 其PAUSE周期比其他框架的CPU长

# 分区表的影响

  - 参考: <http://mysql.taobao.org/monthly/2015/10/03/>
    - 在锁竞争多时, 出现的现象与百胜一致: 
      - 压力出现在 lock_sys的锁 上, 需要频繁进入锁系统进行锁的增减, 进入锁系统需要互斥锁保护
      - CPU高出现在ut_delay上, 在获取互斥锁时进行自旋
      - 结合百胜的CPU架构, 其自旋时间比其他架构CPU要长
  - 结合现场图
    - ![image2021-5-18 18:10:33.png](/assets/01KJBYDB30D5DK9MDJ70R5YFST/image2021-5-18%2018%3A10%3A33.png)
    - 该事务的状态是LOCK WAIT, 持有367个锁结构, 锁heap大小为41168, 其中2个行锁
    - 行锁明显少于锁结构, 怀疑与分区表相关, 该分区表的分片数为 365
    - 该update语句的条件命中id, 但该表的主键为(id, promise_time), 分区键为promise_time, MySQL 理应获取所有表的IX锁
    - 如果有大量并发, 那么分区表涉及的锁的数量明显多于普通表
  - 实验
    - 模仿现场表结构, 生成分区表
    - 压测分区表: 

```
~/opt/mysql/5.7.25/bin/mysqlslap --create-schema='test' --query='update t_order set b = 1 where id = 349989792 ' --host=127.0.0.1 --port=5725 --user=msandbox --password=msandbox --no-drop --iterations=1 --concurrency=100 --number-of-queries=1000000
```

  
  

      - 获得perf top:
        - ![image2021-5-18 18:16:6.png](/assets/01KJBYDB30D5DK9MDJ70R5YFST/image2021-5-18%2018%3A16%3A6.png)
    - 压测普通表

```
~/opt/mysql/5.7.25/bin/mysqlslap --create-schema='test' --query='update t_order PARTITION (p138) set b = 1 where id = 349989792 ' --host=127.0.0.1 --port=5725 --user=msandbox --password=msandbox --no-drop --iterations=1 --concurrency=100 --number-of-queries=1000000
```

      - 获得perf top:
        - ![image2021-5-18 18:17:1.png](/assets/01KJBYDB30D5DK9MDJ70R5YFST/image2021-5-18%2018%3A17%3A1.png)
