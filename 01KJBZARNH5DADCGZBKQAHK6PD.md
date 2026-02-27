---
title: 20240722 - MySQL crash coredump 分析
confluence_page_id: 3145813
created_at: 2024-07-22T06:15:01+00:00
updated_at: 2024-07-22T08:16:04+00:00
---

用户现场MySQL crash, 取回了 coredump. 报错: signal 11

bt: 

```
(gdb) where
#0  0x00007f0b6c3aa9b1 in pthread_kill () from /lib64/libpthread.so.0
#1  0x0000000000802a33 in handle_fatal_signal (sig=11) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/signal_handler.cc:227
#2  <signal handler called>
#3  0x0000000001339ffb in buf_page_optimistic_get (rw_latch=1, block=0x7f0874fead98, modify_clock=0, file=0x17e1f40 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0sel.cc", line=3766, mtr=0x7f0369be1c60)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/buf/buf0buf.cc:4746
#4  0x0000000001311a7b in btr_cur_optimistic_latch_leaves (block=0x7f0874fead98, modify_clock=0, latch_mode=0x7f0369be1ac8, cursor=0x7f027000b118, file=0x17e1f40 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0sel.cc",
    line=3766, mtr=0x7f0369be1c60) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0cur.cc:436
#5  0x0000000001322bcb in btr_pcur_restore_position_func (latch_mode=1, cursor=0x7f027000b118, file=0x17e1f40 "/export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0sel.cc", line=3766, mtr=0x7f0369be1c60)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/btr/btr0pcur.cc:293
#6  0x0000000001280ecd in sel_restore_position_for_mysql (same_user_rec=0x7f0369be2cd8, pcur=0x7f027000b118, moves_up=1, mtr=0x7f0369be1c60, latch_mode=1) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0sel.cc:3766
#7  0x00000000012883ee in row_search_mvcc (buf=0x7f02700a5470 "+y|j,", mode=PAGE_CUR_G, prebuilt=0x7f027000af08, match_mode=0, direction=1) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/row/row0sel.cc:5112
#8  0x0000000001195073 in ha_innobase::general_fetch (this=0x7f02700a3020, buf=0x7f02700a5470 "+y|j,", direction=1, match_mode=0) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/innobase/handler/ha_innodb.cc:9066
#9  0x00000000008501a1 in handler::ha_rnd_next (this=0x7f02700a3020, buf=0x7f02700a5470 "+y|j,") at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/handler.cc:2962
#10 0x0000000000846fff in find_all_keys (found_rows=0x7f0369be3420, pq=0x0, tempfile=0x7f0369be3050, chunk_file=0x7f0369be2f30, fs_info=0x7f0369be3200, qep_tab=0x7f02701ee0a8, param=0x7f0369be3170)
    at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/filesort.cc:986
#11 filesort (thd=0x7f0270098070, filesort=0x7f0270096a10, sort_positions=false, examined_rows=0x7f0369be3428, found_rows=0x7f0369be3420, returned_rows=0x7f0369be3418) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/filesort.cc:430
#12 0x0000000000d23d8f in create_sort_index (tab=0x7f02701ee0a8, join=<optimized out>, thd=0x7f0270098070) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_executor.cc:3712
#13 QEP_TAB::sort_table (this=0x7f02701ee0a8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_executor.cc:2625
#14 0x0000000000d23efe in join_init_read_record (tab=0x7f02701ee0a8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_executor.cc:2491
#15 0x0000000000d25a4a in sub_select (join=0x7f02701ec5c8, qep_tab=0x7f02701ee0a8, end_of_records=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_executor.cc:1284
#16 0x0000000000d24f7a in do_select (join=0x7f02701ec5c8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_executor.cc:957
#17 JOIN::exec (this=0x7f02701ec5c8) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_executor.cc:206
#18 0x0000000000d902a0 in handle_query (thd=0x7f0270098070, lex=0x7f027009a1e0, result=0x7f02700966d8, added_options=1, removed_options=0) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_select.cc:191
#19 0x0000000000d50683 in execute_sqlcom_select (thd=0x7f0270098070, all_tables=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_parse.cc:5155
#20 0x0000000000d5413a in mysql_execute_command (thd=0x7f0270098070, first_level=true) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_parse.cc:2826
#21 0x0000000000d55c85 in mysql_parse (thd=0x7f0270098070, parser_state=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_parse.cc:5584
#22 0x0000000000d5726e in dispatch_command (thd=0x7f0270098070, com_data=0x7f0369be4de0, command=COM_QUERY) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_parse.cc:1491
#23 0x0000000000d57f84 in do_command (thd=0x7f0270098070) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/sql_parse.cc:1032
#24 0x0000000000e2b2ac in handle_connection (arg=<optimized out>) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/sql/conn_handler/connection_handler_per_thread.cc:313
#25 0x0000000001103f14 in pfs_spawn_thread (arg=0x37ccba60) at /export/home/pb2/build/sb_0-39489236-1591101761.33/mysql-5.7.31/storage/perfschema/pfs.cc:2197
#26 0x00007f0b6c3a5e25 in start_thread () from /lib64/libpthread.so.0
#27 0x00007f0b6ae6234d in clone () from /lib64/libc.so.6
``` 

代码: buf_page_optimistic_get函数, 位置: [buf0buf.cc](<http://buf0buf.cc>):4746

![image2024-7-22 13:30:6.png](/assets/01KJBZARNH5DADCGZBKQAHK6PD/image2024-7-22%2013%3A30%3A6.png)

汇编: 

![image2024-7-22 13:30:44.png](/assets/01KJBZARNH5DADCGZBKQAHK6PD/image2024-7-22%2013%3A30%3A44.png)

寄存器: 

```
(gdb) info reg
rax            0x0	0
rbx            0x7f0874fead98	139674299313560
rcx            0x3ad8000	61702144
rdx            0x1	1
rsi            0x1fffffff	536870911
rdi            0x7f0874feae20	139674299313696
rbp            0x7f0369be19f0	0x7f0369be19f0
rsp            0x7f0369be1940	0x7f0369be1940
r8             0x3e8	1000
r9             0x7f0369be1c60	139652635696224
r10            0x7f0270096190	139648446325136
r11            0x7f0b6aeee6d0	139687015409360
r12            0x7f0874feae20	139674299313696
r13            0x7f0874feaef0	139674299313904
r14            0x2013f38	33636152
r15            0x2013f40	33636160
rip            0x1339ffb	0x1339ffb <buf_page_optimistic_get(unsigned long, buf_block_t*, unsigned long, char const*, unsigned long, mtr_t*)+427>
eflags         0x282	[ SF IF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0
gs             0x0	0
``` 

($rbp-0x90) 存放的内容是0:

```
(gdb) p/x ($rbp-0x90)
$9 = 0x7f0369be1960
(gdb) p/x *0x7f0369be1960
$10 = 0x0
``` 

($rbx + 0x120) 存放的内容是0:

```
(gdb) p/x ($rbx + 0x120)
$4 = 0x7f0874feaeb8
(gdb) p/x *($rbx + 0x120)
$5 = 0x0
``` 

崩溃位置的汇编中, mov和cmp都不会报错: 

```
0x0000000001339ff4 <+420>:	mov    -0x90(%rbp),%rax
=> 0x0000000001339ffb <+427>:	cmp    %rax,0x120(%rbx)
   0x000000000133a002 <+434>:	je     0x133a2a0 <buf_page_optimistic_get(unsigned long, buf_block_t*, unsigned long, char const*, unsigned long, mtr_t*)+1104>
``` 

可能性: 

  1. MySQL释放了此处内存 (但没有修改内容), 通过my_malloc中的魔数来确认? (需要做个实验)
  2. 操作系统突然限制了内存? 

查看页的LRU列表, 对比未发现内存内容异常: 

![image2024-7-22 16:15:40.png](/assets/01KJBZARNH5DADCGZBKQAHK6PD/image2024-7-22%2016%3A15%3A40.png)
