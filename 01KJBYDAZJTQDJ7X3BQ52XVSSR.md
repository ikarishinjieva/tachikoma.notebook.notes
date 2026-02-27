---
title: 20210525 - 临时磁盘表, INFORMATION_SCHEMA.FILES 和 `performance_schema`.`file_summary_by_instance` 观测不一致
confluence_page_id: 754141
created_at: 2021-05-25T07:47:08+00:00
updated_at: 2021-05-25T10:08:16+00:00
---

# 造表语句

```
create table tt(a int primary key, c varchar(1024));
 
insert into tt values(1, repeat('c', 1024));
 
insert into tt select (select count(1) from tt) + a, c from tt;
``` 

# 观测方式1

```
mysql [localhost:5725] {msandbox} ((none)) > SELECT FILE_NAME, TABLESPACE_NAME, ENGINE, INITIAL_SIZE, TOTAL_EXTENTS*EXTENT_SIZE        AS TotalSizeBytes, DATA_FREE, MAXIMUM_SIZE FROM INFORMATION_SCHEMA.FILES        WHERE TABLESPACE_NAME = 'innodb_temporary';
+-----------+------------------+--------+--------------+----------------+-----------+--------------+
| FILE_NAME | TABLESPACE_NAME  | ENGINE | INITIAL_SIZE | TotalSizeBytes | DATA_FREE | MAXIMUM_SIZE |
+-----------+------------------+--------+--------------+----------------+-----------+--------------+
| ./ibtmp1  | innodb_temporary | InnoDB |     12582912 |      213909504 | 207618048 |         NULL |
+-----------+------------------+--------+--------------+----------------+-----------+--------------+
1 row in set (0.00 sec)
``` 

  - data_free的来源  

    - fsp_get_available_space_in_free_extents
      - 对 space->free_len 进行部分调整

# 观测方式2

```
mysql [localhost:5725] {msandbox} ((none)) > select * from `performance_schema`.`file_summary_by_instance` where file_name like '%ibtmp1%'\G
*************************** 1. row ***************************
                FILE_NAME: /root/sandboxes/msb_5_7_25/data/ibtmp1
               EVENT_NAME: wait/io/file/innodb/innodb_data_file
    OBJECT_INSTANCE_BEGIN: 140034516365632
               COUNT_STAR: 12365
           SUM_TIMER_WAIT: 116526631880
           MIN_TIMER_WAIT: 4988614
           AVG_TIMER_WAIT: 9423729
           MAX_TIMER_WAIT: 134610454
               COUNT_READ: 11395
           SUM_TIMER_READ: 108549189482
           MIN_TIMER_READ: 4988614
           AVG_TIMER_READ: 9525965
           MAX_TIMER_READ: 134610454
 SUM_NUMBER_OF_BYTES_READ: 186695680
              COUNT_WRITE: 970
          SUM_TIMER_WRITE: 7977442398
          MIN_TIMER_WRITE: 6534724
          AVG_TIMER_WRITE: 8224132
          MAX_TIMER_WRITE: 50140054
SUM_NUMBER_OF_BYTES_WRITE: 15892480
               COUNT_MISC: 0
           SUM_TIMER_MISC: 0
           MIN_TIMER_MISC: 0
           AVG_TIMER_MISC: 0
           MAX_TIMER_MISC: 0
1 row in set (0.00 sec)
``` 

# 实验

使得tt表包含131072行数据, 复制到tt2

```
mysql [localhost:5725] {msandbox} (test) > select count(1) from tt;
+----------+
| count(1) |
+----------+
|   131072 |
+----------+
1 row in set (0.92 sec)

mysql [localhost:5725] {msandbox} (test) > create table tt2 like tt;
Query OK, 0 rows affected (0.01 sec)

mysql [localhost:5725] {msandbox} (test) > insert into tt2 select * from tt;
Query OK, 131072 rows affected (3.83 sec)
Records: 131072  Duplicates: 0  Warnings: 0
``` 

每次测试的预清理工作: 

```
mysql [localhost:5725] {msandbox} (test) > truncate table tt; insert into tt select * from tt2;
Query OK, 0 rows affected (0.03 sec)

Query OK, 131072 rows affected (3.74 sec)
Records: 131072  Duplicates: 0  Warnings: 0

mysql [localhost:5725] {msandbox} (test) > truncate table `performance_schema`.`file_summary_by_instance`;
Query OK, 0 rows affected (0.00 sec)

mysql [localhost:5725] {msandbox} (test) > SELECT FILE_NAME, TABLESPACE_NAME, ENGINE, INITIAL_SIZE, TOTAL_EXTENTS*EXTENT_SIZE        AS TotalSizeBytes, DATA_FREE, MAXIMUM_SIZE FROM INFORMATION_SCHEMA.FILES        WHERE TABLESPACE_NAME = 'innodb_temporary';
+-----------+------------------+--------+--------------+----------------+-----------+--------------+
| FILE_NAME | TABLESPACE_NAME  | ENGINE | INITIAL_SIZE | TotalSizeBytes | DATA_FREE | MAXIMUM_SIZE |
+-----------+------------------+--------+--------------+----------------+-----------+--------------+
| ./ibtmp1  | innodb_temporary | InnoDB |     12582912 |      213909504 | 207618048 |         NULL |
+-----------+------------------+--------+--------------+----------------+-----------+--------------+
1 row in set (0.00 sec)
 
mysql [localhost:5725] {msandbox} (test) > select * from `performance_schema`.`file_summary_by_instance` where file_name like '%ibtmp1%'\G
*************************** 1. row ***************************
                FILE_NAME: /root/sandboxes/msb_5_7_25/data/ibtmp1
               EVENT_NAME: wait/io/file/innodb/innodb_data_file
    OBJECT_INSTANCE_BEGIN: 140034516365632
               COUNT_STAR: 0
           SUM_TIMER_WAIT: 0
           MIN_TIMER_WAIT: 0
           AVG_TIMER_WAIT: 0
           MAX_TIMER_WAIT: 0
               COUNT_READ: 0
           SUM_TIMER_READ: 0
           MIN_TIMER_READ: 0
           AVG_TIMER_READ: 0
           MAX_TIMER_READ: 0
 SUM_NUMBER_OF_BYTES_READ: 0
              COUNT_WRITE: 0
          SUM_TIMER_WRITE: 0
          MIN_TIMER_WRITE: 0
          AVG_TIMER_WRITE: 0
          MAX_TIMER_WRITE: 0
SUM_NUMBER_OF_BYTES_WRITE: 0
               COUNT_MISC: 0
           SUM_TIMER_MISC: 0
           MIN_TIMER_MISC: 0
           AVG_TIMER_MISC: 0
           MAX_TIMER_MISC: 0
1 row in set (0.01 sec)
``` 

并重启服务器, 使得临时表空间回到初始值

测试: 

在free_tmp_table处设置单线程断点, 卡住回收临时表的操作: 

初始状态: 

```
mysql [localhost:5725] {msandbox} ((none)) > SELECT FILE_NAME, TABLESPACE_NAME, ENGINE, INITIAL_SIZE, TOTAL_EXTENTS*EXTENT_SIZE        AS TotalSizeBytes, DATA_FREE, MAXIMUM_SIZE FROM INFORMATION_SCHEMA.FILES        WHERE TABLESPACE_NAME = 'innodb_temporary'\G
*************************** 1. row ***************************
      FILE_NAME: ./ibtmp1
TABLESPACE_NAME: innodb_temporary
         ENGINE: InnoDB
   INITIAL_SIZE: 12582912
 TotalSizeBytes: 683671552
      DATA_FREE: 671088640
   MAXIMUM_SIZE: NULL
1 row in set (7.20 sec)

mysql [localhost:5725] {msandbox} ((none)) > select * from `performance_schema`.`file_summary_by_instance` where file_name like '%ibtmp1%'\G
*************************** 1. row ***************************
                FILE_NAME: /root/sandboxes/msb_5_7_25/data/ibtmp1
               EVENT_NAME: wait/io/file/innodb/innodb_data_file
    OBJECT_INSTANCE_BEGIN: 140093234777152
               COUNT_STAR: 0
           SUM_TIMER_WAIT: 0
           MIN_TIMER_WAIT: 0
           AVG_TIMER_WAIT: 0
           MAX_TIMER_WAIT: 0
               COUNT_READ: 0
           SUM_TIMER_READ: 0
           MIN_TIMER_READ: 0
           AVG_TIMER_READ: 0
           MAX_TIMER_READ: 0
 SUM_NUMBER_OF_BYTES_READ: 0
              COUNT_WRITE: 0
          SUM_TIMER_WRITE: 0
          MIN_TIMER_WRITE: 0
          AVG_TIMER_WRITE: 0
          MAX_TIMER_WRITE: 0
SUM_NUMBER_OF_BYTES_WRITE: 0
               COUNT_MISC: 0
           SUM_TIMER_MISC: 0
           MIN_TIMER_MISC: 0
           AVG_TIMER_MISC: 0
           MAX_TIMER_MISC: 0
1 row in set (0.00 sec)
``` 

临时表回收前的状态: 

```
mysql [localhost:5725] {msandbox} ((none)) > SELECT FILE_NAME, TABLESPACE_NAME, ENGINE, INITIAL_SIZE, TOTAL_EXTENTS*EXTENT_SIZE        AS TotalSizeBytes, DATA_FREE, MAXIMUM_SIZE FROM INFORMATION_SCHEMA.FILES        WHERE TABLESPACE_NAME = 'innodb_temporary'\G
*************************** 1. row ***************************
      FILE_NAME: ./ibtmp1
TABLESPACE_NAME: innodb_temporary
         ENGINE: InnoDB
   INITIAL_SIZE: 12582912
 TotalSizeBytes: 683671552
      DATA_FREE: 517996544
   MAXIMUM_SIZE: NULL
1 row in set (12.67 sec)

mysql [localhost:5725] {msandbox} ((none)) > select * from `performance_schema`.`file_summary_by_instance` where file_name like '%ibtmp1%'\G                *************************** 1. row ***************************
                FILE_NAME: /root/sandboxes/msb_5_7_25/data/ibtmp1
               EVENT_NAME: wait/io/file/innodb/innodb_data_file
    OBJECT_INSTANCE_BEGIN: 140093234777152
               COUNT_STAR: 18766
           SUM_TIMER_WAIT: 741701844268
           MIN_TIMER_WAIT: 5038572
           AVG_TIMER_WAIT: 39523572
           MAX_TIMER_WAIT: 3952979184
               COUNT_READ: 9381
           SUM_TIMER_READ: 304755307912
           MIN_TIMER_READ: 5038572
           AVG_TIMER_READ: 32486124
           MAX_TIMER_READ: 3477666368
 SUM_NUMBER_OF_BYTES_READ: 153698304
              COUNT_WRITE: 9385
          SUM_TIMER_WRITE: 436946536356
          MIN_TIMER_WRITE: 9139152
          AVG_TIMER_WRITE: 46557676
          MAX_TIMER_WRITE: 3952979184
SUM_NUMBER_OF_BYTES_WRITE: 153763840
               COUNT_MISC: 0
           SUM_TIMER_MISC: 0
           MIN_TIMER_MISC: 0
           AVG_TIMER_MISC: 0
           MAX_TIMER_MISC: 0
1 row in set (0.00 sec)
``` 

临时表回收后的状态: 

```
mysql [localhost:5725] {msandbox} ((none)) > SELECT FILE_NAME, TABLESPACE_NAME, ENGINE, INITIAL_SIZE, TOTAL_EXTENTS*EXTENT_SIZE        AS TotalSizeBytes, DATA_FREE, MAXIMUM_SIZE FROM INFORMATION_SCHEMA.FILES        WHERE TABLESPACE_NAME = 'innodb_temporary'\G
*************************** 1. row ***************************
      FILE_NAME: ./ibtmp1
TABLESPACE_NAME: innodb_temporary
         ENGINE: InnoDB
   INITIAL_SIZE: 12582912
 TotalSizeBytes: 683671552
      DATA_FREE: 671088640
   MAXIMUM_SIZE: NULL
1 row in set (7.41 sec)

mysql [localhost:5725] {msandbox} ((none)) > select * from `performance_schema`.`file_summary_by_instance` where file_name like '%ibtmp1%'\G                *************************** 1. row ***************************
                FILE_NAME: /root/sandboxes/msb_5_7_25/data/ibtmp1
               EVENT_NAME: wait/io/file/innodb/innodb_data_file
    OBJECT_INSTANCE_BEGIN: 140093234777152
               COUNT_STAR: 18777
           SUM_TIMER_WAIT: 743139626328
           MIN_TIMER_WAIT: 5038572
           AVG_TIMER_WAIT: 39577076
           MAX_TIMER_WAIT: 3952979184
               COUNT_READ: 9385
           SUM_TIMER_READ: 305717663460
           MIN_TIMER_READ: 5038572
           AVG_TIMER_READ: 32574740
           MAX_TIMER_READ: 3477666368
 SUM_NUMBER_OF_BYTES_READ: 153763840
              COUNT_WRITE: 9392
          SUM_TIMER_WRITE: 437421962868
          MIN_TIMER_WRITE: 9139152
          AVG_TIMER_WRITE: 46573560
          MAX_TIMER_WRITE: 3952979184
SUM_NUMBER_OF_BYTES_WRITE: 153878528
               COUNT_MISC: 0
           SUM_TIMER_MISC: 0
           MIN_TIMER_MISC: 0
           AVG_TIMER_MISC: 0
           MAX_TIMER_MISC: 0
1 row in set (0.00 sec)
``` 

结论: 两种观测手段的观测值差异不大

重新实验, 使得临时表空间extend, 得出的结论相同 (差异3M左右)

# 其他信息

表空间auto_extend的大小由sys_tablespace_auto_extend_increment决定, 默认值为64M (4096页), 无法调整

扩展临时表空间的堆栈: 

```
(gdb) bt
#0  fil_space_extend (space=0x3e90be8, size=10240)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/fil/fil0fil.cc:4983
#1  0x0000000001c6312e in fsp_try_extend_data_file (space=0x3e90be8,
    header=0x7f69f4850026 "", mtr=0x7f69fc143e10)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/fsp/fsp0fsp.cc:1524
#2  0x0000000001c696d4 in fsp_reserve_free_extents (
    n_reserved=0x7f69fc143d50, space_id=1498, n_ext=3,
    alloc_type=FSP_NORMAL, mtr=0x7f69fc143e10, n_pages=2)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/fsp/fsp0fsp.cc:3440
#3  0x0000000001b976b0 in btr_cur_pessimistic_insert (flags=0,
    cursor=0x7f69fc144640, offsets=0x7f69fc144770, heap=0x7f69fc144778,
    entry=0x7f69a9c2ace0, rec=0x7f69fc144768, big_rec=0x7f69fc144780,
    n_ext=0, thr=0x7f69a8bf9b50, mtr=0x7f69fc143e10)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/btr/btr0cur.cc:3388
#4  0x0000000001a619af in row_ins_clust_index_entry_low (flags=0, mode=33,
    index=0x7f69a8bfd470, n_uniq=1, entry=0x7f69a9c2ace0, n_ext=0,
    thr=0x7f69a8bf9b50, dup_chk_only=false)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0ins.cc:2636
#5  0x0000000001a63b69 in row_ins_clust_index_entry (index=0x7f69a8bfd470,
    entry=0x7f69a9c2ace0, thr=0x7f69a8bf9b50, n_ext=0, dup_chk_only=false)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0ins.cc:3368
#6  0x0000000001a63eec in row_ins_index_entry (index=0x7f69a8bfd470,
    entry=0x7f69a9c2ace0, thr=0x7f69a8bf9b50)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0ins.cc:3475
#7  0x0000000001a64446 in row_ins_index_entry_step (node=0x7f69a8bf94e0,
    thr=0x7f69a8bf9b50)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0ins.cc:3625
#8  0x0000000001a647d7 in row_ins (node=0x7f69a8bf94e0, thr=0x7f69a8bf9b50)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0ins.cc:3767
#9  0x0000000001a64dff in row_ins_step (thr=0x7f69a8bf9b50)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0ins.cc:3952
#10 0x0000000001a831c7 in row_insert_for_mysql_using_ins_graph (
    mysql_rec=0x7f69a8c1d700 "\376H\004\002", prebuilt=0x7f69a8bf8f40)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0mysql.cc:1738
#11 0x0000000001a836f6 in row_insert_for_mysql (
    mysql_rec=0x7f69a8c1d700 "\376H\004\002", prebuilt=0x7f69a8bf8f40)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0mysql.cc:1861
#12 0x000000000192b0c5 in ha_innobase::write_row (this=0x7f69a8be7720,
    record=0x7f69a8c1d700 "\376H\004\002")
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:7598
#13 0x0000000000f66d84 in handler::ha_write_row (this=0x7f69a8be7720,
    buf=0x7f69a8c1d700 "\376H\004\002")
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:8048
#14 0x00000000017ca9fd in write_record (thd=0x7f69a8000dd0,
    table=0x7f69a8bf66c0, info=0x7f69a8c089a8, update=0x7f69a8c08a20)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_insert.cc:1873
#15 0x00000000017cbf01 in Query_result_insert::send_data (
    this=0x7f69a8c08960, values=...)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_insert.cc:2271
#16 0x0000000001552224 in end_send (join=0x7f69a8c08f40,
    qep_tab=0x7f69a8c209c0, end_of_records=false)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_executor.cc:2914
#17 0x000000000154eeb1 in evaluate_join_record (join=0x7f69a8c08f40,
    qep_tab=0x7f69a8c20848)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_executor.cc:1645
#18 0x0000000001556f23 in QEP_tmp_table::end_send (this=0x7f69a8c094b8)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_executor.cc:4705
#19 0x000000000154dea4 in sub_select_op (join=0x7f69a8c08f40,
    qep_tab=0x7f69a8c20848, end_of_records=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_executor.cc:1069
#20 0x000000000154dfdc in sub_select (join=0x7f69a8c08f40,
    qep_tab=0x7f69a8c206d0, end_of_records=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_executor.cc:1226
#21 0x000000000154db56 in do_select (join=0x7f69a8c08f40)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_executor.cc:952
#22 0x000000000154b9e2 in JOIN::exec (this=0x7f69a8c08f40)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_executor.cc:199
#23 0x00000000015e7b2a in handle_query (thd=0x7f69a8000dd0, lex=
    0x7f69a8002f68, result=0x7f69a8c08960, added_options=1342177280,
    removed_options=0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_select.cc:184
#24 0x00000000017ceb53 in Sql_cmd_insert_select::execute (
    this=0x7f69a8c088e8, thd=0x7f69a8000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_insert.cc:3218
#25 0x0000000001597b85 in mysql_execute_command (thd=0x7f69a8000dd0,
    first_level=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:3596
#26 0x000000000159db38 in mysql_parse (thd=0x7f69a8000dd0,
    parser_state=0x7f69fc147670)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:5570
#27 0x0000000001592714 in dispatch_command (thd=0x7f69a8000dd0,
    com_data=0x7f69fc147de0, command=COM_QUERY)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1484
#28 0x0000000001591559 in do_command (thd=0x7f69a8000dd0)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1025
#29 0x00000000016c7f75 in handle_connection (arg=0x41a3a30)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/conn_handler/connection_handler_per_thread.cc:306
#30 0x0000000001d72280 in pfs_spawn_thread (arg=0x3f76a10)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/perfschema/pfs.cc:2190
#31 0x00007f6a089186db in start_thread (arg=0x7f69fc148700)
    at pthread_create.c:463
#32 0x00007f6a072b171f in clone ()
    at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

释放临时表空间的堆栈: 

```
(gdb) bt
#0  fsp_free_extent (page_id=..., page_size=..., mtr=0x7f69fc143f70)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/fsp/fsp0fsp.cc:2139
#1  0x0000000001c6b439 in fseg_free_extent (seg_inode=0x7f69f3e258f2 "", space=1501, page_size=..., page=37760, ahi=true, mtr=0x7f69fc143f70)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/fsp/fsp0fsp.cc:3823
#2  0x0000000001c6b885 in fseg_free_step (header=0x7f69f124004a "", ahi=true, mtr=0x7f69fc143f70)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/fsp/fsp0fsp.cc:3890
#3  0x0000000001b7ba6e in btr_free_but_not_root (block=0x7f69efa96680, log_mode=MTR_LOG_NO_REDO)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/btr/btr0btr.cc:1152
#4  0x0000000001b7bd57 in btr_free (page_id=..., page_size=...)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/btr/btr0btr.cc:1223
#5  0x0000000001c01c11 in dict_drop_index_tree_in_mem (index=0x7f69a8bf75b0, page_no=35)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/dict/dict0crea.cc:1187
#6  0x0000000001a89854 in row_drop_table_for_mysql (name=0x7f69fc145760 "tmp/#sql_3590_0", trx=0x7f69fe282d08, drop_db=false, nonatomic=true,
    handler=0x7f69a8c0a7d0) at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/row/row0mysql.cc:4662
#7  0x00000000019343c5 in ha_innobase::delete_table (this=0x7f69a8bf3cb0, name=0x7f69a8c0ca10 "/root/sandboxes/msb_5_7_25/tmp/#sql_3590_0")
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/innobase/handler/ha_innodb.cc:12583
#8  0x0000000000f5f63b in handler::drop_table (this=0x7f69a8bf3cb0, name=0x7f69a8c0ca10 "/root/sandboxes/msb_5_7_25/tmp/#sql_3590_0")
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:4490
#9  0x0000000000f603d6 in handler::ha_drop_table (this=0x7f69a8bf3cb0, name=0x7f69a8c0ca10 "/root/sandboxes/msb_5_7_25/tmp/#sql_3590_0")
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/handler.cc:4949
#10 0x0000000001643252 in free_tmp_table (thd=0x7f69a8000dd0, entry=0x7f69a8c0ba90)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_tmp_table.cc:2416
#11 0x00000000015ed6bf in QEP_TAB::cleanup (this=0x7f69a8c20848) at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_select.cc:2413
#12 0x00000000015e906a in JOIN::destroy (this=0x7f69a8c08f40) at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_select.cc:938
#13 0x000000000164b705 in st_select_lex::cleanup (this=0x7f69a80059e0, full=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_union.cc:1078
#14 0x000000000164b256 in st_select_lex_unit::cleanup (this=0x7f69a8005cc8, full=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_union.cc:923
#15 0x000000000159c5be in mysql_execute_command (thd=0x7f69a8000dd0, first_level=true)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:4996
#16 0x000000000159db38 in mysql_parse (thd=0x7f69a8000dd0, parser_state=0x7f69fc147670)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:5570
#17 0x0000000001592714 in dispatch_command (thd=0x7f69a8000dd0, com_data=0x7f69fc147de0, command=COM_QUERY)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1484
#18 0x0000000001591559 in do_command (thd=0x7f69a8000dd0) at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/sql_parse.cc:1025
#19 0x00000000016c7f75 in handle_connection (arg=0x41a3a30)
    at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/sql/conn_handler/connection_handler_per_thread.cc:306
#20 0x0000000001d72280 in pfs_spawn_thread (arg=0x3f76a10) at /export/home/pb2/build/sb_0-32013917-1545390211.74/mysql-5.7.25/storage/perfschema/pfs.cc:2190
#21 0x00007f6a089186db in start_thread (arg=0x7f69fc148700) at pthread_create.c:463
#22 0x00007f6a072b171f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```
