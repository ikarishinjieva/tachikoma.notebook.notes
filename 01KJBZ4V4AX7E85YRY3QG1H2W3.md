---
title: 20240103 - 博般MySQL crash
confluence_page_id: 2589946
created_at: 2024-01-03T03:04:37+00:00
updated_at: 2024-01-03T06:37:21+00:00
---

# 原始工单

数据库版本: MySQL 5.7.31

崩溃两次

<https://support.actionsky.com/service_desk/browse/SHAI-10843>

<https://support.actionsky.com/service_desk/browse/SHAI-10993>

# 崩溃堆栈

第一次: 

```
/data/mysql/base/5.7.31/bin/mysqld(my_print_stacktrace+0x35)[0xf8e005]
/data/mysql/base/5.7.31/bin/mysqld(handle_fatal_signal+0x4b9)[0x802aa9]
/lib64/libpthread.so.0(+0xf5e0)[0x7f09ee2475e0]

/data/mysql/base/5.7.31/bin/mysqld(_Z22trx_undo_free_preparedP5trx_t+0x1e)[0x7f18d4]
Breakpoint 5 at 0x7f18d4: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/trx/trx0undo.cc, line 2049.
 

/data/mysql/base/5.7.31/bin/mysqld(_Z31btr_cur_optimistic_latch_leavesP11buf_block_tmPmP9btr_cur_tPKcmP5mtr_t+0x6b)[0x1311a7b]
Breakpoint 6 at 0x1311a7b: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0cur.cc, line 436.
 

/data/mysql/base/5.7.31/bin/mysqld(_Z30btr_pcur_restore_position_funcmP10btr_pcur_tPKcmP5mtr_t+0x18b)[0x1322bcb]
Breakpoint 7 at 0x1322bcb: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0pcur.cc, line 293.
 

/data/mysql/base/5.7.31/bin/mysqld[0x1280ecd]
/data/mysql/base/5.7.31/bin/mysqld(_Z15row_search_mvccPh15page_cur_mode_tP14row_prebuilt_tmm+0x2b1e)[0x12883ee]
/data/mysql/base/5.7.31/bin/mysqld(_ZN11ha_innobase13general_fetchEPhjj+0x1a3)[0x1195073]
/data/mysql/base/5.7.31/bin/mysqld(_ZN7handler11ha_rnd_nextEPh+0x154)[0x850214]
/data/mysql/base/5.7.31/bin/mysqld(_Z8filesortP3THDP8FilesortbPyS3_S3_+0x143f)[0x846fff]
/data/mysql/base/5.7.31/bin/mysqld(_ZN7QEP_TAB10sort_tableEv+0x10f)[0xd23d8f]
/data/mysql/base/5.7.31/bin/mysqld(_Z21join_init_read_recordP7QEP_TAB+0x2e)[0xd23efe]
/data/mysql/base/5.7.31/bin/mysqld(_Z10sub_selectP4JOINP7QEP_TABb+0x2ba)[0xd25a4a]
/data/mysql/base/5.7.31/bin/mysqld(_ZN4JOIN4execEv+0x27a)[0xd24f7a]
/data/mysql/base/5.7.31/bin/mysqld(_Z12handle_queryP3THDP3LEXP12Query_resultyy+0x250)[0xd902a0]
/data/mysql/base/5.7.31/bin/mysqld[0xd50683]
/data/mysql/base/5.7.31/bin/mysqld(_Z21mysql_execute_commandP3THDb+0x37da)[0xd5413a]
/data/mysql/base/5.7.31/bin/mysqld(_Z11mysql_parseP3THDP12Parser_state+0x395)[0xd55c85]
/data/mysql/base/5.7.31/bin/mysqld(_Z16dispatch_commandP3THDPK8COM_DATA19enum_server_command+0x153e)[0xd5726e]
/data/mysql/base/5.7.31/bin/mysqld(_Z10do_commandP3THD+0x194)[0xd57f84]
/data/mysql/base/5.7.31/bin/mysqld(handle_connection+0x2ac)[0xe2b2ac]
/data/mysql/base/5.7.31/bin/mysqld(pfs_spawn_thread+0x174)[0x1103f14]
/lib64/libpthread.so.0(+0x7e25)[0x7f09ee23fe25]
/lib64/libc.so.6(clone+0x6d)[0x7f09eccfc34d]
``` 

第二次: 

```
stack_bottom = 7f1ba1b9eea8 thread_stack 0x40000
/data/mysql/base/5.7.31/bin/mysqld(my_print_stacktrace+0x35)[0xf8e005]
/data/mysql/base/5.7.31/bin/mysqld(handle_fatal_signal+0x4b9)[0x802aa9]
/lib64/libpthread.so.0(+0xf5e0)[0x7f23c056e5e0]
 
/data/mysql/base/5.7.31/bin/mysqld(_Z22dtuple_convert_big_recP12dict_index_tP5upd_tP8dtuple_tPm+0x28)[0x135b548]
Breakpoint 3 at 0x135b548: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/data/data0data.cc, line 599.

/data/mysql/base/5.7.31/bin/mysqld(_ZN7BtrBulk6insertEP8dtuple_tm+0x48a)[0x133003a]
Breakpoint 2 at 0x133003a: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0bulk.cc, line 848.

/data/mysql/base/5.7.31/bin/mysqld(_Z31btr_cur_optimistic_latch_leavesP11buf_block_tmPmP9btr_cur_tPKcmP5mtr_t+0x6b)[0x1311a7b]
Breakpoint 1 at 0x1311a7b: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0cur.cc, line 436.

/data/mysql/base/5.7.31/bin/mysqld(_Z30btr_pcur_restore_position_funcmP10btr_pcur_tPKcmP5mtr_t+0x18b)[0x1322bcb]
Breakpoint 4 at 0x1322bcb: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0pcur.cc, line 293.
 

/data/mysql/base/5.7.31/bin/mysqld[0x1280ecd]
/data/mysql/base/5.7.31/bin/mysqld(_Z15row_search_mvccPh15page_cur_mode_tP14row_prebuilt_tmm+0x2b1e)[0x12883ee]
/data/mysql/base/5.7.31/bin/mysqld(_ZN11ha_innobase13general_fetchEPhjj+0x1a3)[0x1195073]
/data/mysql/base/5.7.31/bin/mysqld(_ZN7handler11ha_rnd_nextEPh+0xe1)[0x8501a1]
/data/mysql/base/5.7.31/bin/mysqld(_Z13rr_sequentialP11READ_RECORD+0x1c)[0xcb7b6c]
/data/mysql/base/5.7.31/bin/mysqld(_Z10sub_selectP4JOINP7QEP_TABb+0x2d6)[0xd25a66]
/data/mysql/base/5.7.31/bin/mysqld(_ZN4JOIN4execEv+0x27a)[0xd24f7a]
/data/mysql/base/5.7.31/bin/mysqld(_Z12handle_queryP3THDP3LEXP12Query_resultyy+0x250)[0xd902a0]
/data/mysql/base/5.7.31/bin/mysqld[0xd50683]
/data/mysql/base/5.7.31/bin/mysqld(_Z21mysql_execute_commandP3THDb+0x37da)[0xd5413a]
/data/mysql/base/5.7.31/bin/mysqld(_Z11mysql_parseP3THDP12Parser_state+0x395)[0xd55c85]
/data/mysql/base/5.7.31/bin/mysqld(_Z16dispatch_commandP3THDPK8COM_DATA19enum_server_command+0x153e)[0xd5726e]
/data/mysql/base/5.7.31/bin/mysqld(_Z10do_commandP3THD+0x194)[0xd57f84]
/data/mysql/base/5.7.31/bin/mysqld(handle_connection+0x2ac)[0xe2b2ac]
/data/mysql/base/5.7.31/bin/mysqld(pfs_spawn_thread+0x174)[0x1103f14]
/lib64/libpthread.so.0(+0x7e25)[0x7f23c0566e25]
/lib64/libc.so.6(clone+0x6d)[0x7f23bf02334d]Trying to get some variables.
``` 

# 现象1 - 堆栈缺失

通过disas诊断 0x1311a7b 所在函数

两次崩溃都在相同的堆栈层次有堆栈丢失: 

```
第一次崩溃: 
 
/data/mysql/base/5.7.31/bin/mysqld(_Z22trx_undo_free_preparedP5trx_t+0x1e)[0x7f18d4]
Breakpoint 5 at 0x7f18d4: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/trx/trx0undo.cc, line 2049.
  
-> 丢失堆栈?
 
/data/mysql/base/5.7.31/bin/mysqld(_Z31btr_cur_optimistic_latch_leavesP11buf_block_tmPmP9btr_cur_tPKcmP5mtr_t+0x6b)[0x1311a7b]
Breakpoint 6 at 0x1311a7b: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0cur.cc, line 436.
- btr_cur_optimistic_latch_leaves
  
 
/data/mysql/base/5.7.31/bin/mysqld(_Z30btr_pcur_restore_position_funcmP10btr_pcur_tPKcmP5mtr_t+0x18b)[0x1322bcb]
Breakpoint 7 at 0x1322bcb: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0pcur.cc, line 293.
- btr_pcur_restore_position_func

---

第二次崩溃: 
/data/mysql/base/5.7.31/bin/mysqld(_Z22dtuple_convert_big_recP12dict_index_tP5upd_tP8dtuple_tPm+0x28)[0x135b548]
Breakpoint 3 at 0x135b548: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/data/data0data.cc, line 599.
 
/data/mysql/base/5.7.31/bin/mysqld(_ZN7BtrBulk6insertEP8dtuple_tm+0x48a)[0x133003a]
Breakpoint 2 at 0x133003a: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0bulk.cc, line 848.
 
-> 丢失堆栈?

/data/mysql/base/5.7.31/bin/mysqld(_Z31btr_cur_optimistic_latch_leavesP11buf_block_tmPmP9btr_cur_tPKcmP5mtr_t+0x6b)[0x1311a7b]
Breakpoint 1 at 0x1311a7b: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0cur.cc, line 436.
 
 
/data/mysql/base/5.7.31/bin/mysqld(_Z30btr_pcur_restore_position_funcmP10btr_pcur_tPKcmP5mtr_t+0x18b)[0x1322bcb]
Breakpoint 4 at 0x1322bcb: file /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0pcur.cc, line 293.

``` 

在本地MySQL制造崩溃, 看一下堆栈是否会丢失

```
指定位置: 
(gdb) bt
#0  rw_lock_x_lock_func_nowait (lock=0x0, file_name=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", line=2875)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/include/sync0rw.ic:504
#1  0x000000000133a27f in pfs_rw_lock_x_lock_func_nowait (line=2875, file_name=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", lock=0x7ff4b3d0a290)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/include/sync0rw.ic:757
#2  buf_page_optimistic_get (rw_latch=2, block=0x7ff4b3d0a208, modify_clock=0, file=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", line=2875, mtr=0x7ff4a57f7fb0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/buf/buf0buf.cc:4730
#3  0x0000000001311a7b in btr_cur_optimistic_latch_leaves (block=0x7ff4b3d0a208, modify_clock=0, latch_mode=0x7ff4a57f7ef8, cursor=0x7ff470007490, file=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc",
    line=2875, mtr=0x7ff4a57f7fb0) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0cur.cc:436
#4  0x0000000001322bcb in btr_pcur_restore_position_func (latch_mode=2, cursor=0x7ff470007490, file=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", line=2875, mtr=0x7ff4a57f7fb0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0pcur.cc:293
#5  0x0000000001299d85 in row_upd_clust_step (node=0x7ff470006878, thr=0x7ff4700082c8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc:2875
#6  0x000000000129b95f in row_upd (node=0x7ff470006878, thr=0x7ff4700082c8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc:3054
#7  0x000000000129bc53 in row_upd_step (thr=0x7ff4700082c8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc:3200
#8  0x000000000122f808 in que_thr_step (thr=0x7ff4700082c8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1039
#9  que_run_threads_low (thr=0x7ff4700082c8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1119
#10 que_run_threads (thr=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1159
#11 0x000000000122ff9e in que_eval_sql (info=0x7ff4700011e8,
    sql=0x17f73e0 "PROCEDURE INDEX_STATS_SAVE () IS\nBEGIN\nDELETE FROM \"mysql/innodb_index_stats\"\nWHERE\ndatabase_name = :database_name AND\ntable_name = :table_name AND\nindex_name = :index_name AND\nstat_name = :stat_name;"..., reserve_dict_mutex=0,
    trx=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1236
#12 0x000000000138c7eb in dict_stats_exec_sql (pinfo=0x7ff4700011e8, sql=<optimized out>, trx=0x7ff4c292eae0) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats.cc:319
#13 0x000000000138ca78 in dict_stats_save_index_stat (index=0x7ff470000f50, last_update=1704257225, stat_name=0x17f84ea "n_leaf_pages", stat_value=1, sample_size=0x0, stat_description=0x17f7c60 "Number of leaf pages in the index", trx=0x7ff4c292eae0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats.cc:2348
#14 0x000000000138f547 in dict_stats_save (table_orig=<optimized out>, only_for_index=0x0) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats.cc:2524
#15 0x00000000013903b2 in dict_stats_update (table=0x7ff4840166d8, stats_upd_option=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats.cc:3127
#16 0x0000000001392b0a in dict_stats_process_entry_from_recalc_pool () at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats_bg.cc:363
#17 dict_stats_thread (arg=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats_bg.cc:452
#18 0x00007ff4ccfc46db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#19 0x00007ff4cb95d71f in clone () from /lib/x86_64-linux-gnu/libc.so.6
 
 
将lock设置为空
 
(gdb) set lock=0
``` 

gdb中的崩溃堆栈: 

```
(gdb) bt
#0  rw_lock_x_lock_func_nowait (lock=0x0,
    file_name=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc",
    line=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/include/sync0rw.ic:503
#1  0x000000000133a27f in pfs_rw_lock_x_lock_func_nowait (line=2875,
    file_name=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc",
    lock=0x7f86dfcfede0) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/include/sync0rw.ic:757
#2  buf_page_optimistic_get (rw_latch=2, block=0x7f86dfcfed58, modify_clock=0,
    file=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", line=2875,
    mtr=0x7f86bfffd310) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/buf/buf0buf.cc:4730
#3  0x0000000001311a7b in btr_cur_optimistic_latch_leaves (block=0x7f86dfcfed58, modify_clock=0, latch_mode=0x7f86bfffd258,
    cursor=0x7f8698002498,
    file=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", line=2875,
    mtr=0x7f86bfffd310) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0cur.cc:436
#4  0x0000000001322bcb in btr_pcur_restore_position_func (latch_mode=2, cursor=0x7f8698002498,
    file=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", line=2875,
    mtr=0x7f86bfffd310) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0pcur.cc:293
#5  0x0000000001299d85 in row_upd_clust_step (node=0x7f8698001dc0, thr=0x7f86980030b0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc:2875
#6  0x000000000129b95f in row_upd (node=0x7f8698001dc0, thr=0x7f86980030b0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc:3054
#7  0x000000000129bc53 in row_upd_step (thr=0x7f86980030b0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc:3200
#8  0x000000000122f808 in que_thr_step (thr=0x7f86980030b0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1039
#9  que_run_threads_low (thr=0x7f86980030b0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1119
#10 que_run_threads (thr=<optimized out>)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1159
#11 0x000000000122ff9e in que_eval_sql (info=0x7f86980011e8,
    sql=0x17f7b08 "PROCEDURE TABLE_STATS_SAVE () IS\nBEGIN\nDELETE FROM \"mysql/innodb_table_stats\"\nWHERE\ndatabase_name = :database_name AND\ntable_name = :table_name;\nINSERT INTO \"mysql/innodb_table_stats\"\nVALUES\n(\n:databa"..., reserve_dict_mutex=0,
    trx=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/que/que0que.cc:1236
#12 0x000000000138c88c in dict_stats_exec_sql (pinfo=0x7f86980011e8,
    sql=0x17f7b08 "PROCEDURE TABLE_STATS_SAVE () IS\nBEGIN\nDELETE FROM \"mysql/innodb_table_stats\"\nWHERE\ndatabase_name = :database_name AND\ntable_name = :table_name;\nINSERT INTO \"mysql/innodb_table_stats\"\nVALUES\n(\n:databa"..., trx=0x7f86ed0f2e70)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats.cc:319
#13 0x000000000138ecf6 in dict_stats_save (table_orig=<optimized out>, only_for_index=0x0)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats.cc:2424
#14 0x00000000013903b2 in dict_stats_update (table=0x7f86a40107e8, stats_upd_option=<optimized out>)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats.cc:3127
#15 0x0000000001392b0a in dict_stats_process_entry_from_recalc_pool ()
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats_bg.cc:363
#16 dict_stats_thread (arg=<optimized out>)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/dict/dict0stats_bg.cc:452
#17 0x00007f86f77886db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#18 0x00007f86f612171f in clone () from /lib/x86_64-linux-gnu/libc.so.6
 
 
(gdb) info frame 0
Stack frame at 0x7f86bfffd0d0:
 rip = 0x13327ed in rw_lock_x_lock_func_nowait
    (/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/include/sync0rw.ic:503); saved rip = 0x133a27f
 called by frame at 0x7f86bfffd190
 source language c++.
 Arglist at 0x7f86bfffd258, args: lock=0x0,
    file_name=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc",
    line=<optimized out>
 Locals at 0x7f86bfffd258, Previous frame's sp is 0x7f86bfffd0d0
 Saved registers:
  rbx at 0x7f86bfffd0a8, rbp at 0x7f86bfffd0c0, r12 at 0x7f86bfffd0b0, r13 at 0x7f86bfffd0b8, rip at 0x7f86bfffd0c8
 
(gdb) info frame 1
Stack frame at 0x7f86bfffd190:
 rip = 0x133a27f in pfs_rw_lock_x_lock_func_nowait
    (/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/include/sync0rw.ic:757); saved rip = 0x1311a7b
 inlined into frame 2, caller of frame at 0x7f86bfffd0d0
 source language c++.
 Arglist at unknown address.
 Locals at unknown address, Previous frame's sp is 0x7f86bfffd0d0
 Saved registers:
  rbx at 0x7f86bfffd0a8, rbp at 0x7f86bfffd0c0, r12 at 0x7f86bfffd0b0, r13 at 0x7f86bfffd0b8, rip at 0x7f86bfffd0c8
 
(gdb) info frame 2
Stack frame at 0x7f86bfffd190:
 rip = 0x133a27f in buf_page_optimistic_get
    (/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/buf/buf0buf.cc:4730); saved rip = 0x1311a7b
 called by frame at 0x7f86bfffd220, caller of frame at 0x7f86bfffd190
 source language c++.
 Arglist at 0x7f86bfffd180, args: rw_latch=2, block=0x7f86dfcfed58, modify_clock=0,
    file=0x17e2dd8 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0upd.cc", line=2875,
    mtr=0x7f86bfffd310
 Locals at 0x7f86bfffd180, Previous frame's sp is 0x7f86bfffd190
 Saved registers:
  rbx at 0x7f86bfffd158, rbp at 0x7f86bfffd180, r12 at 0x7f86bfffd160, r13 at 0x7f86bfffd168, r14 at 0x7f86bfffd170,
  r15 at 0x7f86bfffd178, rip at 0x7f86bfffd188
``` 

error log中的堆栈信息: 

```
04:51:25 UTC - mysqld got signal 11 ;
This could be because you hit a bug. It is also possible that this binary
or one of the libraries it was linked against is corrupt, improperly built,
or misconfigured. This error can also be caused by malfunctioning hardware.
Attempting to collect some information that could help diagnose the problem.
As this is a crash and something is definitely wrong, the information
collection process might fail.

key_buffer_size=8388608
read_buffer_size=131072
max_used_connections=1
max_threads=151
thread_count=1
connection_count=1
It is possible that mysqld could use up to
key_buffer_size + (read_buffer_size + sort_buffer_size)*max_threads = 68196 K  bytes of memory
Hope that's ok; if not, decrease some variables in the equation.

Thread pointer: 0x0
Attempting backtrace. You can use the following information to find out
where mysqld died. If you see no messages after this, something went
terribly wrong...
stack_bottom = 0 thread_stack 0x40000
/root/opt/mysql/5.7.31/bin/mysqld(my_print_stacktrace+0x35)[0xf8e005]
/root/opt/mysql/5.7.31/bin/mysqld(handle_fatal_signal+0x4b9)[0x802aa9]
/lib/x86_64-linux-gnu/libpthread.so.0(+0x12980)[0x7ff4ccfcf980]
/root/opt/mysql/5.7.31/bin/mysqld[0x13327e3]
/root/opt/mysql/5.7.31/bin/mysqld(_Z23buf_page_optimistic_getmP11buf_block_tmPKcmP5mtr_t+0x42f)[0x133a27f]
/root/opt/mysql/5.7.31/bin/mysqld(_Z31btr_cur_optimistic_latch_leavesP11buf_block_tmPmP9btr_cur_tPKcmP5mtr_t+0x6b)[0x1311a7b]
/root/opt/mysql/5.7.31/bin/mysqld(_Z30btr_pcur_restore_position_funcmP10btr_pcur_tPKcmP5mtr_t+0x18b)[0x1322bcb]
/root/opt/mysql/5.7.31/bin/mysqld[0x1299d85]
/root/opt/mysql/5.7.31/bin/mysqld(_Z7row_updP10upd_node_tP9que_thr_t+0x6f)[0x129b95f]
/root/opt/mysql/5.7.31/bin/mysqld(_Z12row_upd_stepP9que_thr_t+0x103)[0x129bc53]
/root/opt/mysql/5.7.31/bin/mysqld(_Z15que_run_threadsP9que_thr_t+0x5e8)[0x122f808]
/root/opt/mysql/5.7.31/bin/mysqld(_Z12que_eval_sqlP11pars_info_tPKcmP5trx_t+0x6e)[0x122ff9e]
/root/opt/mysql/5.7.31/bin/mysqld[0x138c7eb]
/root/opt/mysql/5.7.31/bin/mysqld[0x138ca78]
/root/opt/mysql/5.7.31/bin/mysqld[0x138f547]
/root/opt/mysql/5.7.31/bin/mysqld(_Z17dict_stats_updateP12dict_table_t23dict_stats_upd_option_t+0xa92)[0x13903b2]
/root/opt/mysql/5.7.31/bin/mysqld(dict_stats_thread+0x70a)[0x1392b0a]
/lib/x86_64-linux-gnu/libpthread.so.0(+0x76db)[0x7ff4ccfc46db]
/lib/x86_64-linux-gnu/libc.so.6(clone+0x3f)[0x7ff4cb95d71f]
``` 

未见堆栈丢失. 与生产现象不符

怀疑是OS的backtrace问题, 生产的服务器版本是: 

![image2024-1-3 14:37:9.png](/assets/01KJBZ4V4AX7E85YRY3QG1H2W3/image2024-1-3%2014%3A37%3A9.png)

找到相同版本的OS进行复现
