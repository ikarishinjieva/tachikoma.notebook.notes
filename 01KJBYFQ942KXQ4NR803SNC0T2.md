---
title: 20211230 - 国债MySQL 8.0 insert性能与MySQL 5.7有差异
confluence_page_id: 1573529
created_at: 2021-12-30T05:56:21+00:00
updated_at: 2022-02-08T08:54:07+00:00
---

# 场景

表结构

```
drop table student;
CREATE TABLE `student` (
  `STUID` int NOT NULL,
  PRIMARY KEY (`STUID`)
);
 
delimiter $$
create procedure a()
begin
set @i =1;
while @i <= 10000 do
insert into student(STUID) values(@i);
set @i = @i+1;
end while;
commit;
end $$
delimiter ;
``` 

call a(), 会发现8.0性能低于5.7

# 探索

生成8.0和5.7的火焰图

```
perf record -F 99 -p $(ps aux | grep 'msb_8_0_25/dat[a]' | awk '{print $2}') -g -- sleep 15
perf script | ./stackcollapse-perf.pl > out.perf-folded
./flamegraph.pl out.perf-folded > perf.8.svg

perf record -F 99 -p $(ps aux | grep 'msb_5_7_26/dat[a]' | awk '{print $2}') -g -- sleep 15
perf script | ./stackcollapse-perf.pl > out.perf-folded
./flamegraph.pl out.perf-folded > perf.5.svg
``` 

## 查看8.0的火焰图

![image2021-12-30 12:39:43.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2021-12-30%2012%3A39%3A43.png)

## sql_log_bin=0, 消除binlog的影响

8.0

![image2021-12-30 12:43:31.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2021-12-30%2012%3A43%3A31.png)

对比5.7的火焰图: 

![image2021-12-30 13:52:15.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2021-12-30%2013%3A52%3A15.png)

## open_table的差异??

8.0

![image2021-12-30 13:53:46.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2021-12-30%2013%3A53%3A46.png)

5.7 

![image2021-12-30 13:54:11.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2021-12-30%2013%3A54%3A11.png)

## 使用内存映射磁盘

```
mount -t tmpfs -o size=4096m tmpfs /mnt/ramdisk
``` 

8.0

![image2021-12-30 14:22:24.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2021-12-30%2014%3A22%3A24.png)

5.7 

![image2021-12-30 14:22:51.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2021-12-30%2014%3A22%3A51.png)

## 消除log_writer中的CRC计算

```
 set global innodb_checksum_algorithm=none;
``` 

分别截取10s on-CPU火焰图, 发现两张图几乎一致

意味着10s内的计算量一致

# 观察off-cpu图

```
(offcputime-bpfcc -df -p $(ps aux | grep 'msb_5_7_26/dat[a]' | awk '{print $2}') 10 > offcpu.5) && (./flamegraph.pl --color=io --title="Off-CPU Time Flame Graph" --countname=us < offcpu.5 > offcpu.5.svg)

(offcputime-bpfcc -df -p $(ps aux | grep 'msb_8_0_25/dat[a]' | awk '{print $2}') 10 > offcpu.5) && (./flamegraph.pl --color=io --title="Off-CPU Time Flame Graph" --countname=us < offcpu.8 > offcpu.8.svg)
``` 

5.7 

![image2022-1-1 21:15:6.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-1%2021%3A15%3A6.png)

8.0

![image2022-1-1 21:15:21.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-1%2021%3A15%3A21.png)

两张图几乎一致

关闭所有不必要的功能

```
innodb_flush_method=O_DSYNC
performance_schema=OFF
innodb_log_writer_threads=OFF
disable_log_bin
``` 

包括 redo刷盘, performance_schema, binlog.

5.7仍优于8.0

# 观察perf-stat

```
perf stat -d -p $(ps aux | grep 'msb_5_7_26/dat[a]' | awk '{print $2}') -- sleep 5
 
perf stat -d -p $(ps aux | grep 'msb_8_0_25/dat[a]' | awk '{print $2}') -- sleep 5
``` 

5.7

![image2022-1-1 21:57:17.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-1%2021%3A57%3A17.png)

8.0

![image2022-1-1 21:57:30.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-1%2021%3A57%3A30.png)

# 观察page fault

```
(perf record -e page-faults -p $(ps aux | grep 'msb_8_0_25/dat[a]' | awk '{print $2}') -g -- sleep 10) && (perf script > out.stacks) && (./stackcollapse-perf.pl < out.stacks | ./flamegraph.pl --color=mem  --title="Page Faults" --countname="pages" > page_fault.8.svg)

(perf record -e page-faults -p $(ps aux | grep 'msb_5_7_26/dat[a]' | awk '{print $2}') -g -- sleep 10) && (perf script > out.stacks) && (./stackcollapse-perf.pl < out.stacks | ./flamegraph.pl --color=mem  --title="Page Faults" --countname="pages" > page_fault.5.svg)

``` 

5.7 

没有page fault的堆栈

8.0 

![image2022-1-1 22:54:44.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-1%2022%3A54%3A44.png)

两种可能: 

  1. 8.0的内存分配器比5.7的效率低, 检查glibc版本, 未见不同
  2. 8.0的相关工作比5.7更多

猜测为可能2, 监控相同数据量 (1000条) 下, 相关函数调用的次数: 

![image2022-1-1 23:46:50.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-1%2023%3A46%3A50.png)

8.0 监控log_buffer_write, 5.7监控log_write_low, 发现次数差20倍

找到 8.0调用log_buffer_write的次数多的原因, 插入1000条记录: 

  1. 发现调用log_buffer_write次数为 5632
  2. 分布为:   
  
![image2022-1-3 16:43:20.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-3%2016%3A43%3A20.png)

对比5.7调用log_write_low的次数: 

  1. 调用次数为: 243
  2. 分布为:   
  
![image2022-1-3 16:56:12.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-3%2016%3A56%3A12.png)

经过gdb调试, 发现MySQL 5.7的redo log写入存在一个短路通道

![image2022-1-3 22:48:11.png](/assets/01KJBYFQ942KXQ4NR803SNC0T2/image2022-1-3%2022%3A48%3A11.png)

当log的第一个block并未填满时, 使用快通道, 不写入文件

慢通道调用堆栈如下, 快慢通道的分差在frame 3: 

```
414     /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc: No such file or directory.
#0  log_write_low (str=str@entry=0x7f51f7c3c5a0 "\246\033\005", str_len=30) at /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc:414
#1  0x0000000000f9f0c5 in mtr_write_log_t::operator() (this=<synthetic pointer>, block=0x7f51f7c3c5a0) at /usr/src/mysql-5.7.32/storage/innobase/mtr/mtr0mtr.cc:393
#2  dyn_buf_t<512ul>::for_each_block<mtr_write_log_t> (this=<optimized out>, functor=<synthetic pointer>...) at /usr/src/mysql-5.7.32/storage/innobase/include/dyn0buf.h:357
#3  mtr_t::Command::finish_write (this=this@entry=0x7f51f7c3bac0, len=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/mtr/mtr0mtr.cc:830
#4  0x0000000000fa05cb in mtr_t::Command::execute (this=this@entry=0x7f51f7c3bac0) at /usr/src/mysql-5.7.32/storage/innobase/mtr/mtr0mtr.cc:866
#5  0x0000000000fa0939 in mtr_t::commit (this=this@entry=0x7f51f7c3c330) at /usr/src/mysql-5.7.32/storage/innobase/mtr/mtr0mtr.cc:496
#6  0x0000000000ff74e8 in row_ins_clust_index_entry_low (flags=flags@entry=0, mode=<optimized out>, mode@entry=2, index=index@entry=0x375e0d8, n_uniq=n_uniq@entry=1, entry=entry@
entry=0x7f51bc01c9c8, n_ext=n_ext@entry=0, thr=<optimized out>, dup_chk_only=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:2664
#7  0x0000000000ffa7a2 in row_ins_clust_index_entry (index=0x375e0d8, entry=0x7f51bc01c9c8, thr=thr@entry=0x7f51bc03a530, n_ext=n_ext@entry=0, dup_chk_only=dup_chk_only@entry=fal
se) at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3299
#8  0x0000000000ffd10e in row_ins_index_entry (thr=0x7f51bc03a530, entry=<optimized out>, index=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3437
#9  row_ins_index_entry_step (thr=0x7f51bc03a530, node=0x7f51bc03a2f8) at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3587
#10 row_ins (thr=0x7f52022c0750, node=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3725
#11 row_ins_step (thr=thr@entry=0x7f51bc03a530) at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3861
#12 0x000000000100d733 in row_insert_for_mysql_using_ins_graph (mysql_rec=mysql_rec@entry=0x7f51bc00dd88 "\377\343\003", prebuilt=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/row/row0mysql.cc:1746
#13 0x0000000001012194 in row_insert_for_mysql (mysql_rec=mysql_rec@entry=0x7f51bc00dd88 "\377\343\003", prebuilt=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/row/row0mysql.cc:1867
#14 0x0000000000f30db5 in ha_innobase::write_row (this=0x7f51bc00da90, record=0x7f51bc00dd88 "\377\343\003") at /usr/src/mysql-5.7.32/storage/innobase/handler/ha_innodb.cc:7627
#15 0x00000000007d9fe4 in handler::ha_write_row (this=0x7f51bc00da90, buf=0x7f51bc00dd88 "\377\343\003") at /usr/src/mysql-5.7.32/sql/handler.cc:8107
#16 0x0000000000dc5a35 in write_record (thd=thd@entry=0x7f51bc000d40, table=table@entry=0x7f51bc00b970, info=info@entry=0x7f51f7c3cf20, update=update@entry=0x7f51f7c3cfa0) at /usr/src/mysql-5.7.32/sql/sql_insert.cc:1895
#17 0x0000000000dc7401 in Sql_cmd_insert::mysql_insert (this=this@entry=0x7f51bc048990, thd=thd@entry=0x7f51bc000d40, table_list=table_list@entry=0x7f51bc0482f0) at /usr/src/mysql-5.7.32/sql/sql_insert.cc:776
#18 0x0000000000dc8172 in Sql_cmd_insert::execute (this=0x7f51bc048990, thd=0x7f51bc000d40) at /usr/src/mysql-5.7.32/sql/sql_insert.cc:3142
#19 0x0000000000c49531 in mysql_execute_command (thd=thd@entry=0x7f51bc000d40, first_level=first_level@entry=false) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:3606
#20 0x0000000000bce6e0 in sp_instr_stmt::exec_core (this=0x7f51bc038a58, thd=0x7f51bc000d40, nextp=0x7f51f7c3e0dc) at /usr/src/mysql-5.7.32/sql/sp_instr.cc:1026
#21 0x0000000000bd07ed in sp_lex_instr::reset_lex_and_exec_core (this=this@entry=0x7f51bc038a58, thd=thd@entry=0x7f51bc000d40, nextp=nextp@entry=0x7f51f7c3e0dc, open_tables=open_tables@entry=false) at /usr/src/mysql-5.7.32/sql/sp_instr.cc:452
#22 0x0000000000bd149b in sp_lex_instr::validate_lex_and_execute_core (this=this@entry=0x7f51bc038a58, thd=thd@entry=0x7f51bc000d40, nextp=nextp@entry=0x7f51f7c3e0dc, open_tables=open_tables@entry=false) at /usr/src/mysql-5.7.32/sql/sp_instr.cc:754
#23 0x0000000000bd2468 in sp_instr_stmt::execute (this=0x7f51bc038a58, thd=0x7f51bc000d40, nextp=0x7f51f7c3e0dc) at /usr/src/mysql-5.7.32/sql/sp_instr.cc:937
#24 0x0000000000bca06a in sp_head::execute (this=this@entry=0x7f51bc02dc70, thd=thd@entry=0x7f51bc000d40, merge_da_on_success=merge_da_on_success@entry=true) at /usr/src/mysql-5.7.32/sql/sp_head.cc:796
#25 0x0000000000bcdbb4 in sp_head::execute_procedure (this=this@entry=0x7f51bc02dc70, thd=thd@entry=0x7f51bc000d40, args=args@entry=0x7f51bc003268) at /usr/src/mysql-5.7.32/sql/sp_head.cc:1529
#26 0x0000000000c483dc in mysql_execute_command (thd=thd@entry=0x7f51bc000d40, first_level=first_level@entry=true) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:4544
#27 0x0000000000c4e2ad in mysql_parse (thd=thd@entry=0x7f51bc000d40, parser_state=parser_state@entry=0x7f51f7c3f740) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:5584
#28 0x0000000000c4ed86 in dispatch_command (thd=thd@entry=0x7f51bc000d40, com_data=com_data@entry=0x7f51f7c3fda0, command=COM_QUERY) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:1491
#29 0x0000000000c508c0 in do_command (thd=thd@entry=0x7f51bc000d40) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:1032
#30 0x0000000000d10ee0 in handle_connection (arg=arg@entry=0x38797c0) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:313
``` 

想法: 8.0的log_buffer_write可能与5.7的快通道log_reserve_and_write_fast逻辑一致, 都是将session内的redo写入redo log buffer, 问题是: 为什么8.0会触发更多的page fault

重新考虑page fault的出处, 使用mysqld-debug来测试, 发现集中于如下堆栈: 

```
mysqld 31335 5320187.458886:         14 page-faults:
                 456f854 mach_write_to_2 (/root/opt/mysql/8.0.25/bin/mysqld-debug)
                 4573ac2 log_block_set_first_rec_group (/root/opt/mysql/8.0.25/bin/mysqld-debug)
                 4571ac7 log_buffer_write (/root/opt/mysql/8.0.25/bin/mysqld-debug)
                 45ca97c mtr_write_log_t::operator() (/root/opt/mysql/8.0.25/bin/mysqld-debug)
                 45cb246 dyn_buf_t<512ul>::for_each_block<mtr_write_log_t> (/root/opt/mysql/8.0.25/bin/mysqld-debug)
                 45c83a2 mtr_t::Command::execute (/root/opt/mysql/8.0.25/bin/mysqld-debug)
                 45c7b06 mtr_t::commit (/root/opt/mysql/8.0.25/bin/mysqld-debug)
                 4791573 trx_undo_assign_undo (/root/opt/mysql/8.0.25/bin/mysqld-debug)
...
``` 

~~超出认知~~

page fault是由于使用内存模拟了磁盘, 写入磁盘的数据, 会导致内存缺失的信号
