---
title: 20210407 - 同样的压力, 不同机器perf呈现的热点不同
confluence_page_id: 753928
created_at: 2021-04-07T15:09:51+00:00
updated_at: 2021-04-07T15:25:57+00:00
---

# 现象

![image2021-4-7 23:8:0.png](/assets/01KJBYD9SKSCBFSFV4QPZYZ5RX/image2021-4-7%2023%3A8%3A0.png)![image2021-4-7 23:8:7.png](/assets/01KJBYD9SKSCBFSFV4QPZYZ5RX/image2021-4-7%2023%3A8%3A7.png)

# 模拟获得rec_get_offsets_func堆栈

```
(gdb) bt
#0  rec_get_offsets_func (rec=0x7f774c608200 "mysql/plugin", index=0x4ddf4c0, offsets=0x7f7758064be0, n_fields=1,
    file=0x2305e40 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc", line=789, heap=0x7f7758064f00)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001a8cef3 in page_cur_search_with_match_bytes (block=0x7f774b9fc250, index=0x4ddf4c0, tuple=0x7f76fc01b210, mode=PAGE_CUR_GE, iup_matched_fields=0x7f7758065d88,
    iup_matched_bytes=0x7f7758065d80, ilow_matched_fields=0x7f7758065d78, ilow_matched_bytes=0x7f7758065d70, cursor=0x7f77580665d8)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc:789
#2  0x0000000001c1c2c4 in btr_cur_search_to_nth_level (index=0x4ddf4c0, level=0, tuple=0x7f76fc01b210, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f77580665d0, has_search_latch=0,
    file=0x2381380 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0load.cc", line=3073, mtr=0x7f77580660c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0cur.cc:1437
#3  0x0000000001c2e75e in btr_pcur_open_low (index=0x4ddf4c0, level=0, tuple=0x7f76fc01b210, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f77580665d0,
    file=0x2381380 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0load.cc", line=3073, mtr=0x7f77580660c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/include/btr0pcur.ic:465
#4  0x0000000001c2fe82 in btr_pcur_open_on_user_rec_func (index=0x4ddf4c0, tuple=0x7f76fc01b210, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f77580665d0,
    file=0x2381380 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0load.cc", line=3073, mtr=0x7f77580660c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0pcur.cc:603
#5  0x0000000001cb406b in dict_load_table_one (name=..., cached=true, ignore_err=DICT_ERR_IGNORE_NONE, fk_tables=std::deque with 0 elements)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0load.cc:3073
#6  0x0000000001cb3653 in dict_load_table (name=0x7f7758067270 "test/a", cached=true, ignore_err=DICT_ERR_IGNORE_NONE)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0load.cc:2865
#7  0x0000000001c9683e in dict_table_open_on_name (table_name=0x7f7758067270 "test/a", dict_locked=0, try_drop=1, ignore_err=DICT_ERR_IGNORE_NONE)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0dict.cc:1248
#8  0x00000000019b35dd in ha_innobase::open_dict_table (table_name=0x7f770844d798 "./test/a", norm_name=0x7f7758067270 "test/a", is_partition=false, ignore_err=DICT_ERR_IGNORE_NONE)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:6238
#9  0x00000000019b2703 in ha_innobase::open (this=0x7f76fc0112e0, name=0x7f770844d798 "./test/a", mode=2, test_if_locked=2)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:5876
#10 0x0000000000f520ef in handler::ha_open (this=0x7f76fc0112e0, table_arg=0x7f76fc01a7f0, name=0x7f770844d798 "./test/a", mode=2, test_if_locked=2)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:2759
#11 0x000000000167f621 in open_table_from_share (thd=0x7f76fc000dd0, share=0x7f770844d410, alias=0x7f76fc005848 "a", db_stat=39, prgflag=8, ha_open_flags=0, outparam=0x7f76fc01a7f0,
    is_create_table=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/table.cc:3353
#12 0x00000000015056d5 in open_table (thd=0x7f76fc000dd0, table_list=0x7f7758068670, ot_ctx=0x7f7758067df0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:3523
#13 0x0000000001508093 in open_and_process_table (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, tables=0x7f7758068670, counter=0x7f76fc003010, flags=1024, prelocking_strategy=0x7f7758067f20,
    has_prelocking_list=false, ot_ctx=0x7f7758067df0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:5109
#14 0x000000000150917e in open_tables (thd=0x7f76fc000dd0, start=0x7f7758067ee0, counter=0x7f76fc003010, flags=1024, prelocking_strategy=0x7f7758067f20)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:5720
#15 0x000000000150a5b9 in open_tables_for_query (thd=0x7f76fc000dd0, tables=0x7f7758068670, flags=1024) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:6495
#16 0x00000000015f1385 in mysqld_list_fields (thd=0x7f76fc000dd0, table_list=0x7f7758068670, wild=0x7f76fc006178 "")
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_show.cc:1131
#17 0x00000000015896ae in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_FIELD_LIST) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1638
#18 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#19 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#20 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#21 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#22 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

```
#0  rec_get_offsets_func (rec=0x7f76fc00fea2 "testaPRIMARYn_diff_pfx01", index=0x4410210, offsets=0x0, n_fields=4,
    file=0x2364dc8 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0pcur.cc", line=309, heap=0x7f7758065d68)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001c2f497 in btr_pcur_restore_position_func (latch_mode=1, cursor=0x7f76fc01f3d0,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=1372, mtr=0x7f7758065e10)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0pcur.cc:309
#2  0x0000000001b42512 in row_sel_restore_pcur_pos (plan=0x7f76fc01f3c0, mtr=0x7f7758065e10) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:1372
#3  0x0000000001b42e19 in row_sel (node=0x7f76fc01ed08, thr=0x7f76fc020c70) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:1722
#4  0x0000000001b44167 in row_sel_step (thr=0x7f76fc020c70) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:2404
#5  0x0000000001ab7a0b in que_thr_step (thr=0x7f76fc020c70) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1034
#6  0x0000000001ab7d20 in que_run_threads_low (thr=0x7f76fc020c70) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1118
#7  0x0000000001ab7ee2 in que_run_threads (thr=0x7f76fc020c70) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1158
#8  0x0000000001ab819b in que_eval_sql (info=0x7f76fc0236c0,
    sql=0x2386400 "PROCEDURE FETCH_STATS () IS\nfound INT;\nDECLARE FUNCTION fetch_table_stats_step;\nDECLARE FUNCTION fetch_index_stats_step;\nDECLARE CURSOR table_stats_cur IS\n  SELECT\n  n_rows,\n  clustered_index_size,\n  "..., reserve_dict_mutex=1, trx=0x7f775a3ab8c0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1235
#9  0x0000000001cc4fc0 in dict_stats_fetch_from_ps (table=0x7f76fc01b210) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0stats.cc:2993
#10 0x0000000001cc56c1 in dict_stats_update (table=0x7f76fc00f9b0, stats_upd_option=DICT_STATS_FETCH_ONLY_IF_NOT_IN_MEMORY)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0stats.cc:3186
#11 0x00000000019a9e9d in dict_stats_init (table=0x7f76fc00f9b0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/include/dict0stats.ic:173
#12 0x00000000019b2b04 in ha_innobase::open (this=0x7f76fc0112e0, name=0x7f770844d798 "./test/a", mode=2, test_if_locked=2)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:5944
#13 0x0000000000f520ef in handler::ha_open (this=0x7f76fc0112e0, table_arg=0x7f76fc01a7f0, name=0x7f770844d798 "./test/a", mode=2, test_if_locked=2)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:2759
#14 0x000000000167f621 in open_table_from_share (thd=0x7f76fc000dd0, share=0x7f770844d410, alias=0x7f76fc005848 "a", db_stat=39, prgflag=8, ha_open_flags=0, outparam=0x7f76fc01a7f0,
    is_create_table=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/table.cc:3353
#15 0x00000000015056d5 in open_table (thd=0x7f76fc000dd0, table_list=0x7f7758068670, ot_ctx=0x7f7758067df0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:3523
#16 0x0000000001508093 in open_and_process_table (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, tables=0x7f7758068670, counter=0x7f76fc003010, flags=1024, prelocking_strategy=0x7f7758067f20,
    has_prelocking_list=false, ot_ctx=0x7f7758067df0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:5109
#17 0x000000000150917e in open_tables (thd=0x7f76fc000dd0, start=0x7f7758067ee0, counter=0x7f76fc003010, flags=1024, prelocking_strategy=0x7f7758067f20)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:5720
#18 0x000000000150a5b9 in open_tables_for_query (thd=0x7f76fc000dd0, tables=0x7f7758068670, flags=1024) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:6495
#19 0x00000000015f1385 in mysqld_list_fields (thd=0x7f76fc000dd0, table_list=0x7f7758068670, wild=0x7f76fc006178 "")
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_show.cc:1131
#20 0x00000000015896ae in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_FIELD_LIST) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1638
#21 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#22 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#23 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#24 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#25 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

```
#0  rec_get_offsets_func (rec=0x7f774c6fc1a7 "testb", index=0x440e9f0, offsets=0x7f77580630b0,
    n_fields=2,
    file=0x2305e40 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc", line=846, heap=0x7f77580633d0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001a8d091 in page_cur_search_with_match_bytes (block=0x7f774ba082d8, index=0x440e9f0,
    tuple=0x7f76fc93dd18, mode=PAGE_CUR_GE, iup_matched_fields=0x7f7758064258,
    iup_matched_bytes=0x7f7758064250, ilow_matched_fields=0x7f7758064248,
    ilow_matched_bytes=0x7f7758064240, cursor=0x7f76fc93da30)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc:846
#2  0x0000000001c1c2c4 in btr_cur_search_to_nth_level (index=0x440e9f0, level=0,
    tuple=0x7f76fc93dd18, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93da28,
    has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=1335, mtr=0x7f7758064940)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0cur.cc:1437
#3  0x0000000001b3f2ad in btr_pcur_open_with_no_init_func (index=0x440e9f0, tuple=0x7f76fc93dd18,
    mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93da28, has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=1335, mtr=0x7f7758064940)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/include/btr0pcur.ic:521
#4  0x0000000001b423a7 in row_sel_open_pcur (plan=0x7f76fc93da18, search_latch_locked=1,
    mtr=0x7f7758064940)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:1335
#5  0x0000000001b427b5 in row_sel_try_search_shortcut (node=0x7f76fc93d360, plan=0x7f76fc93da18,
    search_latch_locked=1, mtr=0x7f7758064940)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:1495
#6  0x0000000001b42d1d in row_sel (node=0x7f76fc93d360, thr=0x7f76fc940190)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:1684
#7  0x0000000001b44167 in row_sel_step (thr=0x7f76fc940190)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:2404
#8  0x0000000001ab7a0b in que_thr_step (thr=0x7f76fc940190)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1034
#9  0x0000000001ab7d20 in que_run_threads_low (thr=0x7f76fc940190)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1118
#10 0x0000000001ab7ee2 in que_run_threads (thr=0x7f76fc940190)
---Type <return> to continue, or q <return> to quit---
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1158
#11 0x0000000001ab819b in que_eval_sql (info=0x7f76fc0236c0,
    sql=0x2386400 "PROCEDURE FETCH_STATS () IS\nfound INT;\nDECLARE FUNCTION fetch_table_stats_step;\nDECLARE FUNCTION fetch_index_stats_step;\nDECLARE CURSOR table_stats_cur IS\n  SELECT\n  n_rows,\n  clustered_index_size,\n  "..., reserve_dict_mutex=1, trx=0x7f775a3abd08)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/que/que0que.cc:1235
#12 0x0000000001cc4fc0 in dict_stats_fetch_from_ps (table=0x7f76fc9398f0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0stats.cc:2993
#13 0x0000000001cc56c1 in dict_stats_update (table=0x7f76fc020b10,
    stats_upd_option=DICT_STATS_FETCH_ONLY_IF_NOT_IN_MEMORY)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/dict/dict0stats.cc:3186
#14 0x00000000019a9e9d in dict_stats_init (table=0x7f76fc020b10)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/include/dict0stats.ic:173
#15 0x00000000019b2b04 in ha_innobase::open (this=0x7f76fc020690, name=0x7f76fc9386c8 "./test/b",
    mode=2, test_if_locked=2)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:5944
#16 0x0000000000f520ef in handler::ha_open (this=0x7f76fc020690, table_arg=0x7f76fc938ed0,
    name=0x7f76fc9386c8 "./test/b", mode=2, test_if_locked=2)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:2759
#17 0x000000000167f621 in open_table_from_share (thd=0x7f76fc000dd0, share=0x7f76fc938340,
    alias=0x7f76fc006368 "b", db_stat=39, prgflag=8, ha_open_flags=0, outparam=0x7f76fc938ed0,
    is_create_table=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/table.cc:3353
#18 0x00000000015056d5 in open_table (thd=0x7f76fc000dd0, table_list=0x7f76fc006370,
    ot_ctx=0x7f7758066920)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:3523
#19 0x0000000001508093 in open_and_process_table (thd=0x7f76fc000dd0, lex=0x7f76fc002f50,
    tables=0x7f76fc006370, counter=0x7f76fc003010, flags=0, prelocking_strategy=0x7f7758066a50,
    has_prelocking_list=false, ot_ctx=0x7f7758066920)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:5109
#20 0x000000000150917e in open_tables (thd=0x7f76fc000dd0, start=0x7f7758066a10,
    counter=0x7f76fc003010, flags=0, prelocking_strategy=0x7f7758066a50)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:5720
#21 0x000000000150a5b9 in open_tables_for_query (thd=0x7f76fc000dd0, tables=0x7f76fc006370, flags=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_base.cc:6495
#22 0x00000000017bc8b0 in Sql_cmd_insert::mysql_insert (this=0x7f76fc006900, thd=0x7f76fc000dd0,
    table_list=0x7f76fc006370)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_insert.cc:464
#23 0x00000000017c4003 in Sql_cmd_insert::execute (this=0x7f76fc006900, thd=0x7f76fc000dd0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_insert.cc:3105
---Type <return> to continue, or q <return> to quit---
#24 0x000000000158e36a in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:3570
#25 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#26 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#27 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#28 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#29 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#30 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#31 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

```
(gdb) bt
#0  rec_get_offsets_func (rec=0x7f774c7b007d "\200", index=0x7f76fc010a20, offsets=0x7f7758064c50, n_fields=18446744073709551615,
    file=0x23199b0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc", line=2297, heap=0x7f7758064f78)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001ae9eea in row_ins_duplicate_error_in_clust (flags=0, cursor=0x7f7758065860, entry=0x7f76fc936960, thr=0x7f76fc93a0b8, mtr=0x7f7758065030)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc:2297
#2  0x0000000001aea80b in row_ins_clust_index_entry_low (flags=0, mode=2, index=0x7f76fc010a20, n_uniq=1, entry=0x7f76fc936960, n_ext=0, thr=0x7f76fc93a0b8, dup_chk_only=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc:2554
#3  0x0000000001aecb35 in row_ins_clust_index_entry (index=0x7f76fc010a20, entry=0x7f76fc936960, thr=0x7f76fc93a0b8, n_ext=0, dup_chk_only=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc:3329
#4  0x0000000001aed043 in row_ins_index_entry (index=0x7f76fc010a20, entry=0x7f76fc936960, thr=0x7f76fc93a0b8)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc:3465
#5  0x0000000001aed59d in row_ins_index_entry_step (node=0x7f76fc939e60, thr=0x7f76fc93a0b8) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc:3615
#6  0x0000000001aed92e in row_ins (node=0x7f76fc939e60, thr=0x7f76fc93a0b8) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc:3757
#7  0x0000000001aedf56 in row_ins_step (thr=0x7f76fc93a0b8) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0ins.cc:3942
#8  0x0000000001b0ca16 in row_insert_for_mysql_using_ins_graph (mysql_rec=0x7f76fc020980 "\377\001", prebuilt=0x7f76fc9398f0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0mysql.cc:1734
#9  0x0000000001b0cf50 in row_insert_for_mysql (mysql_rec=0x7f76fc020980 "\377\001", prebuilt=0x7f76fc9398f0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0mysql.cc:1858
#10 0x00000000019b61d3 in ha_innobase::write_row (this=0x7f76fc020690, record=0x7f76fc020980 "\377\001")
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:7554
#11 0x0000000000f5ed4e in handler::ha_write_row (this=0x7f76fc020690, buf=0x7f76fc020980 "\377\001") at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:7977
#12 0x00000000017c0423 in write_record (thd=0x7f76fc000dd0, table=0x7f76fc938ed0, info=0x7f7758066be0, update=0x7f7758066b60)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_insert.cc:1871
#13 0x00000000017bd510 in Sql_cmd_insert::mysql_insert (this=0x7f76fc006900, thd=0x7f76fc000dd0, table_list=0x7f76fc006370)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_insert.cc:769
#14 0x00000000017c4003 in Sql_cmd_insert::execute (this=0x7f76fc006900, thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_insert.cc:3105
#15 0x000000000158e36a in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:3570
#16 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#17 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#18 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#19 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#20 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#21 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#22 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

```
#0  rec_get_offsets_func (rec=0x7f774c7b007d "\200", index=0x7f76fc010a20, offsets=0x7f7758065d30, n_fields=18446744073709551615,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=5343, heap=0x7f77580660b0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001b4ab3c in row_search_mvcc (buf=0x7f76fc020980 "\377\001", mode=PAGE_CUR_G, prebuilt=0x7f76fc9398f0, match_mode=0, direction=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:5343
#2  0x00000000019b897c in ha_innobase::index_read (this=0x7f76fc020690, buf=0x7f76fc020980 "\377\001", key_ptr=0x0, key_len=0, find_flag=HA_READ_AFTER_KEY)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:8696
#3  0x00000000019b99de in ha_innobase::index_first (this=0x7f76fc020690, buf=0x7f76fc020980 "\377\001")
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:9113
#4  0x0000000000f54638 in handler::ha_index_first (this=0x7f76fc020690, buf=0x7f76fc020980 "\377\001") at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:3185
#5  0x00000000015470c3 in join_read_first (tab=0x7f76fc0073a0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:2636
#6  0x0000000001543b63 in sub_select (join=0x7f76fc006cc0, qep_tab=0x7f76fc0073a0, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1271
#7  0x00000000015434e8 in do_select (join=0x7f76fc006cc0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:944
#8  0x00000000015413be in JOIN::exec (this=0x7f76fc006cc0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:199
#9  0x00000000015de3fa in handle_query (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, result=0x7f76fc006ae0, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_select.cc:184
#10 0x0000000001593533 in execute_sqlcom_select (thd=0x7f76fc000dd0, all_tables=0x7f76fc006498) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5159
#11 0x000000000158c127 in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:2796
#12 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#13 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#14 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#15 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#16 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#17 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#18 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

select * from b; ?

```
#0  rec_get_offsets_func (rec=0x7f76fc01005d "\200", index=0x7f76fc010a20, offsets=0x0, n_fields=1,
    file=0x2364dc8 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0pcur.cc", line=309, heap=0x7f7758065768)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001c2f497 in btr_pcur_restore_position_func (latch_mode=1, cursor=0x7f76fc939b08,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=3751, mtr=0x7f7758065870)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0pcur.cc:309
#2  0x0000000001b47105 in sel_restore_position_for_mysql (same_user_rec=0x7f7758066108, latch_mode=1, pcur=0x7f76fc939b08, moves_up=1, mtr=0x7f7758065870)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:3751
#3  0x0000000001b49d43 in row_search_mvcc (buf=0x7f76fc020980 "\377\001", mode=PAGE_CUR_G, prebuilt=0x7f76fc9398f0, match_mode=0, direction=1)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:5062
#4  0x00000000019b9699 in ha_innobase::general_fetch (this=0x7f76fc020690, buf=0x7f76fc020980 "\377\001", direction=1, match_mode=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:8996
#5  0x00000000019b98cb in ha_innobase::index_next (this=0x7f76fc020690, buf=0x7f76fc020980 "\377\001")
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:9065
#6  0x0000000000f53f3d in handler::ha_index_next (this=0x7f76fc020690, buf=0x7f76fc020980 "\377\001") at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:3117
#7  0x000000000154712e in join_read_next (info=0x7f76fc0073f0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:2650
#8  0x0000000001543b79 in sub_select (join=0x7f76fc006cc0, qep_tab=0x7f76fc0073a0, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1274
#9  0x00000000015434e8 in do_select (join=0x7f76fc006cc0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:944
#10 0x00000000015413be in JOIN::exec (this=0x7f76fc006cc0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:199
#11 0x00000000015de3fa in handle_query (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, result=0x7f76fc006ae0, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_select.cc:184
#12 0x0000000001593533 in execute_sqlcom_select (thd=0x7f76fc000dd0, all_tables=0x7f76fc006498) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5159
#13 0x000000000158c127 in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:2796
#14 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#15 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#16 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#17 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#18 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#19 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#20 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

select * from b b1, b b2 where b1.b=b2.b order by b1.b limit 1;

```
(gdb) bt
#0  rec_get_offsets_func (rec=0x7f774c7b007d "\200", index=0x7f76fc010a20, offsets=0x7f7758064190, n_fields=1,
    file=0x2305e40 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc", line=846, heap=0x7f77580644b0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001a8d091 in page_cur_search_with_match_bytes (block=0x7f774ba110e0, index=0x7f76fc010a20, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, iup_matched_fields=0x7f7758065338,
    iup_matched_bytes=0x7f7758065330, ilow_matched_fields=0x7f7758065328, ilow_matched_bytes=0x7f7758065320, cursor=0x7f76fc93ea20)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc:846
#2  0x0000000001c1c2c4 in btr_cur_search_to_nth_level (index=0x7f76fc010a20, level=0, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93ea18, has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4059, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0cur.cc:1437
#3  0x0000000001b3f2ad in btr_pcur_open_with_no_init_func (index=0x7f76fc010a20, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93ea18, has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4059, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/include/btr0pcur.ic:521
#4  0x0000000001b47a82 in row_sel_try_search_shortcut_for_mysql (out_rec=0x7f7758065f88, prebuilt=0x7f76fc93e800, offsets=0x7f7758065f48, heap=0x7f7758065f50, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4059
#5  0x0000000001b4965e in row_search_mvcc (buf=0x7f76fc938a80 "", mode=PAGE_CUR_GE, prebuilt=0x7f76fc93e800, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4871
#6  0x00000000019b897c in ha_innobase::index_read (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key_ptr=0x7f76fc93e688 "\001", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:8696
#7  0x0000000000f61a7c in handler::index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.h:2783
#8  0x0000000000f5324e in handler::ha_index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:3031
#9  0x0000000001545ab7 in join_read_key (tab=0x7f76fc93f9e0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:2096
#10 0x0000000001543b63 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f9e0, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1271
#11 0x0000000001544873 in evaluate_join_record (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1639
#12 0x0000000001543c74 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1291
#13 0x00000000015434e8 in do_select (join=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:944
#14 0x00000000015413be in JOIN::exec (this=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:199
#15 0x00000000015de3fa in handle_query (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, result=0x7f76fc007728, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_select.cc:184
#16 0x0000000001593533 in execute_sqlcom_select (thd=0x7f76fc000dd0, all_tables=0x7f76fc006ac8) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5159
#17 0x000000000158c127 in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:2796
#18 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#19 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#20 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#21 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#22 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#23 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#24 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

```
#0  rec_get_offsets_func (rec=0x7f774c7b007d "\200", index=0x7f76fc010a20, offsets=0x7f7758065bd0,
    n_fields=18446744073709551615,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4080, heap=0x7f7758065f50)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001b47b17 in row_sel_try_search_shortcut_for_mysql (out_rec=0x7f7758065f88,
    prebuilt=0x7f76fc93e800, offsets=0x7f7758065f48, heap=0x7f7758065f50, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4080
#2  0x0000000001b4965e in row_search_mvcc (buf=0x7f76fc938a80 "", mode=PAGE_CUR_GE,
    prebuilt=0x7f76fc93e800, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4871
#3  0x00000000019b897c in ha_innobase::index_read (this=0x7f76fc938790, buf=0x7f76fc938a80 "",
    key_ptr=0x7f76fc93e688 "\001", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:8696
#4  0x0000000000f61a7c in handler::index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "",
    key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.h:2783
#5  0x0000000000f5324e in handler::ha_index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "",
    key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:3031
#6  0x0000000001545ab7 in join_read_key (tab=0x7f76fc93f9e0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:2096
#7  0x0000000001543b63 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f9e0,
    end_of_records=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1271
#8  0x0000000001544873 in evaluate_join_record (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1639
#9  0x0000000001543c74 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868,
    end_of_records=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1291
#10 0x00000000015434e8 in do_select (join=0x7f76fc93cf08)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:944
#11 0x00000000015413be in JOIN::exec (this=0x7f76fc93cf08)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:199
#12 0x00000000015de3fa in handle_query (thd=0x7f76fc000dd0, lex=0x7f76fc002f50,
    result=0x7f76fc007728, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_select.cc:184
#13 0x0000000001593533 in execute_sqlcom_select (thd=0x7f76fc000dd0, all_tables=0x7f76fc006ac8)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5159
#14 0x000000000158c127 in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:2796
#15 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670)
---Type <return> to continue, or q <return> to quit---
   me/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#16 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#17 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#18 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#19 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#20 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#21 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

# select * from b b1, b b2 where b1.b=b2.b order by b1.b limit 1; 命中堆栈

包括 buf_page_get_gen 和 rec_get_offsets_func 断点:

```
#0  buf_page_get_gen (page_id=..., page_size=..., rw_latch=1, guess=0x7f774ba110e0, mode=10,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4059, mtr=0x7f77580656c0, dirty_with_no_latch=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/buf/buf0buf.cc:4067
#1  0x0000000001c1b343 in btr_cur_search_to_nth_level (index=0x7f76fc010a20, level=0, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93ea18, has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4059, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0cur.cc:1107
#2  0x0000000001b3f2ad in btr_pcur_open_with_no_init_func (index=0x7f76fc010a20, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93ea18, has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4059, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/include/btr0pcur.ic:521
#3  0x0000000001b47a82 in row_sel_try_search_shortcut_for_mysql (out_rec=0x7f7758065f88, prebuilt=0x7f76fc93e800, offsets=0x7f7758065f48, heap=0x7f7758065f50, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4059
#4  0x0000000001b4965e in row_search_mvcc (buf=0x7f76fc938a80 "", mode=PAGE_CUR_GE, prebuilt=0x7f76fc93e800, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4871
#5  0x00000000019b897c in ha_innobase::index_read (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key_ptr=0x7f76fc93e688 "\001", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:8696
#6  0x0000000000f61a7c in handler::index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.h:2783
#7  0x0000000000f5324e in handler::ha_index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:3031
#8  0x0000000001545ab7 in join_read_key (tab=0x7f76fc93f9e0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:2096
#9  0x0000000001543b63 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f9e0, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1271
#10 0x0000000001544873 in evaluate_join_record (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1639
#11 0x0000000001543c74 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1291
#12 0x00000000015434e8 in do_select (join=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:944
#13 0x00000000015413be in JOIN::exec (this=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:199
#14 0x00000000015de3fa in handle_query (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, result=0x7f76fc007728, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_select.cc:184
#15 0x0000000001593533 in execute_sqlcom_select (thd=0x7f76fc000dd0, all_tables=0x7f76fc006ac8) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5159
#16 0x000000000158c127 in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:2796
#17 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#18 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#19 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#20 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#21 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#22 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#23 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```
```
#0  rec_get_offsets_func (rec=0x7f774c7b007d "\200", index=0x7f76fc010a20, offsets=0x7f7758064190, n_fields=1,
    file=0x2305e40 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc", line=846, heap=0x7f77580644b0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001a8d091 in page_cur_search_with_match_bytes (block=0x7f774ba110e0, index=0x7f76fc010a20, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, iup_matched_fields=0x7f7758065338,
    iup_matched_bytes=0x7f7758065330, ilow_matched_fields=0x7f7758065328, ilow_matched_bytes=0x7f7758065320, cursor=0x7f76fc93ea20)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/page/page0cur.cc:846
#2  0x0000000001c1c2c4 in btr_cur_search_to_nth_level (index=0x7f76fc010a20, level=0, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93ea18, has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4059, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/btr/btr0cur.cc:1437
#3  0x0000000001b3f2ad in btr_pcur_open_with_no_init_func (index=0x7f76fc010a20, tuple=0x7f76fc93ec08, mode=PAGE_CUR_GE, latch_mode=1, cursor=0x7f76fc93ea18, has_search_latch=1,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4059, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/include/btr0pcur.ic:521
#4  0x0000000001b47a82 in row_sel_try_search_shortcut_for_mysql (out_rec=0x7f7758065f88, prebuilt=0x7f76fc93e800, offsets=0x7f7758065f48, heap=0x7f7758065f50, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4059
#5  0x0000000001b4965e in row_search_mvcc (buf=0x7f76fc938a80 "", mode=PAGE_CUR_GE, prebuilt=0x7f76fc93e800, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4871
#6  0x00000000019b897c in ha_innobase::index_read (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key_ptr=0x7f76fc93e688 "\001", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:8696
#7  0x0000000000f61a7c in handler::index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.h:2783
#8  0x0000000000f5324e in handler::ha_index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:3031
#9  0x0000000001545ab7 in join_read_key (tab=0x7f76fc93f9e0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:2096
#10 0x0000000001543b63 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f9e0, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1271
#11 0x0000000001544873 in evaluate_join_record (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1639
#12 0x0000000001543c74 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1291
#13 0x00000000015434e8 in do_select (join=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:944
#14 0x00000000015413be in JOIN::exec (this=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:199
#15 0x00000000015de3fa in handle_query (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, result=0x7f76fc007728, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_select.cc:184
#16 0x0000000001593533 in execute_sqlcom_select (thd=0x7f76fc000dd0, all_tables=0x7f76fc006ac8) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5159
#17 0x000000000158c127 in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:2796
#18 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#19 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#20 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#21 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#22 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#23 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#24 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

```
#0  rec_get_offsets_func (rec=0x7f774c7b007d "\200", index=0x7f76fc010a20, offsets=0x7f7758065bd0, n_fields=18446744073709551615,
    file=0x232e3f0 "/export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc", line=4080, heap=0x7f7758065f50)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/rem/rem0rec.cc:556
#1  0x0000000001b47b17 in row_sel_try_search_shortcut_for_mysql (out_rec=0x7f7758065f88, prebuilt=0x7f76fc93e800, offsets=0x7f7758065f48, heap=0x7f7758065f50, mtr=0x7f77580656c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4080
#2  0x0000000001b4965e in row_search_mvcc (buf=0x7f76fc938a80 "", mode=PAGE_CUR_GE, prebuilt=0x7f76fc93e800, match_mode=1, direction=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/row/row0sel.cc:4871
#3  0x00000000019b897c in ha_innobase::index_read (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key_ptr=0x7f76fc93e688 "\001", key_len=4, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/innobase/handler/ha_innodb.cc:8696
#4  0x0000000000f61a7c in handler::index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.h:2783
#5  0x0000000000f5324e in handler::ha_index_read_map (this=0x7f76fc938790, buf=0x7f76fc938a80 "", key=0x7f76fc93e688 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:3031
#6  0x0000000001545ab7 in join_read_key (tab=0x7f76fc93f9e0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:2096
#7  0x0000000001543b63 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f9e0, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1271
#8  0x0000000001544873 in evaluate_join_record (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1639
#9  0x0000000001543c74 in sub_select (join=0x7f76fc93cf08, qep_tab=0x7f76fc93f868, end_of_records=false) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:1291
#10 0x00000000015434e8 in do_select (join=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:944
#11 0x00000000015413be in JOIN::exec (this=0x7f76fc93cf08) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_executor.cc:199
#12 0x00000000015de3fa in handle_query (thd=0x7f76fc000dd0, lex=0x7f76fc002f50, result=0x7f76fc007728, added_options=0, removed_options=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_select.cc:184
#13 0x0000000001593533 in execute_sqlcom_select (thd=0x7f76fc000dd0, all_tables=0x7f76fc006ac8) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5159
#14 0x000000000158c127 in mysql_execute_command (thd=0x7f76fc000dd0, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:2796
#15 0x00000000015943b9 in mysql_parse (thd=0x7f76fc000dd0, parser_state=0x7f7758068670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#16 0x0000000001588d5e in dispatch_command (thd=0x7f76fc000dd0, com_data=0x7f7758068de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#17 0x0000000001587b61 in do_command (thd=0x7f76fc000dd0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#18 0x00000000016be3c0 in handle_connection (arg=0x7f7708000bb0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#19 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4eccf80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#20 0x00007f7764a0f6db in start_thread (arg=0x7f7758069700) at pthread_create.c:463
#21 0x00007f77633a871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```
