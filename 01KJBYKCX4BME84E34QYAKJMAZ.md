---
title: 20220707 - 观察MySQL对 cgroup IO限制 造成压力的来源
confluence_page_id: 1933409
created_at: 2022-07-07T11:08:38+00:00
updated_at: 2022-07-08T11:17:19+00:00
---

# 目的

如果用cgroup限制MySQL IO, 那么如何分析是MySQL读/写哪些文件对IO造成了压力

# 分析 biosnoop 的输出

biosnoop输出样例: 

```
root@ubuntu:~/sandboxes/msb_8_0_20# biosnoop-bpfcc -d /dev/vda1
TIME(s)        COMM           PID    DISK    T  SECTOR    BYTES   LAT(ms)
0.000000000    ?              0              R  -1        8          0.32
2.016049000    ?              0              R  -1        8          0.36
2.615445000    mysql          18779  vda     W  24647728  12288      4.35
3.872201000    jbd2/vda1-8    377    vda     W  2355360   61440      0.35
3.872568000    jbd2/vda1-8    377    vda     W  2355480   4096       0.15
4.032018000    ?              0              R  -1        8          0.33
6.048063000    ?              0              R  -1        8          0.36
8.064028000    ?              0              R  -1        8          0.33
``` 

## 如何查找扇区对应的文件

查找 扇区24647728 的对应文件 X

参考文献: <https://mellowhost.com/blog/identifying-file-inode-by-sector-block-number-in-linux.html>

找到 /dev/vda1 的偏移量: 

```
root@ubuntu:~/sandboxes/msb_8_0_20# fdisk -lu /dev/vda
Disk /dev/vda: 320 GiB, 343597383680 bytes, 671088640 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 848AD1B1-1D5F-401B-A434-EA84F4486685

Device      Start       End   Sectors   Size Type
/dev/vda1  227328 671088606 670861279 319.9G Linux filesystem
/dev/vda14   2048     10239      8192     4M BIOS boot
/dev/vda15  10240    227327    217088   106M EFI System

Partition table entries are not in disk order.
root@ubuntu:~/sandboxes/msb_8_0_20#
``` 

/dev/vda1的偏移量 = 227328

扇区大小 = 512 字节

X在/dev/vda1的偏移量 = 24647728 - 227328 = 24420400

检测block size: 

```
root@ubuntu:~/sandboxes/msb_8_0_20# tune2fs -l /dev/vda1 | grep Block
Block count:              83857659
Block size:               4096
Blocks per group:         32768
root@ubuntu:~/sandboxes/msb_8_0_20#
``` 

块大小 = 4096

X的块号 = 24420400/4096*512 = 3052550

找到块号对应的inode号: 

```
root@ubuntu:~/sandboxes/msb_8_0_20# debugfs /dev/vda1
debugfs 1.44.1 (24-Mar-2018)
debugfs:  icheck 3052550
Block	Inode number
3052550	1149612
``` 

找到inode号对应的文件名: 

```
root@ubuntu:~/sandboxes/msb_8_0_20# debugfs /dev/vda1
debugfs 1.44.1 (24-Mar-2018)
debugfs:  icheck 3052550
Block	Inode number
3052550	1149612
debugfs:  ncheck 1149612
Inode	Pathname
1149612	/root/sandboxes/msb_8_0_20/.mysql_history
``` 

## 如何查找文件对应的扇区

参考: <https://unix.stackexchange.com/questions/106802/what-command-do-i-use-to-see-the-start-and-end-block-of-a-file-in-the-file-syste>

找到文件的inode号: 

```
root@ubuntu:~/sandboxes/msb_8_0_20/data# ls -i binlog.000003
1033887 binlog.000003
``` 

找到文件的extent列表: 

```
root@ubuntu:~/sandboxes/msb_8_0_20/data# debugfs /dev/vda1
debugfs 1.44.1 (24-Mar-2018)
debugfs:  stat <1033887>
 
--- 以下为输出
 
Inode: 1033887   Type: regular    Mode:  0640   Flags: 0x80000
Generation: 1635333115    Version: 0x00000000:00000001
User:     0   Group:     0   Project:     0   Size: 8538975
File ACL: 0
Links: 1   Blockcount: 16680
Fragment:  Address: 0    Number: 0    Size: 0
 ctime: 0x62c06216:8677d400 -- Sat Jul  2 15:19:50 2022
 atime: 0x62c0621a:c7516400 -- Sat Jul  2 15:19:54 2022
 mtime: 0x62c06216:8677d400 -- Sat Jul  2 15:19:50 2022
crtime: 0x62c060d8:791ddc00 -- Sat Jul  2 15:14:32 2022
Size of extra inode fields: 32
Inode checksum: 0xfcd18bb9
EXTENTS:
(0):57775370, (1-511):31329280-31329790, (512-1023):31327232-31327743, (1024-2084):31331328-31332388
``` 

extent段 (均为块号): 

  - (0):57775370, 
  - (1-511):31329280-31329790, 
  - (512-1023):31327232-31327743, 
  - (1024-2084):31331328-31332388

# biosnoop的输出

使用一块独立的磁盘, 对MySQL进行sysbench压力, 并通过cgroup限制 blkio.throttle.write_iops_device

发现其中有三个进程在进行写: 

  - mysql
  - kworker
  - jbd2

通过stackcount分析写入堆栈 (submit_bio): 

```
/usr/share/bcc/tools/stackcount -P -Tdi 1 submit_bio
``` 

kworker的堆栈: 

```
  submit_bio
  ext4_writepages
  do_writepages
  __writeback_single_inode
  writeback_sb_inodes
  __writeback_inodes_wb
  wb_writeback
  wb_workfn
  process_one_work
  worker_thread
  kthread
  ret_from_fork
    --
    kworker/u16:0 [19787]
``` 

jbd2的堆栈: 

```
  submit_bio
  submit_bh
  journal_submit_commit_record
  jbd2_journal_commit_transaction
  kjournald2
  kthread
  ret_from_fork
    --
    jbd2/dm-1-8 [20359]
``` 

通过stackcount分析IO限流堆栈 (blk_throtl_bio) (参考: <https://elixir.bootlin.com/linux/v4.15/source/block/blk-throttle.c>): 
    
    
    kworker堆栈: 

```
  blk_throtl_bio
  generic_make_request
  __map_bio
  __clone_and_map_simple_bio
  __split_and_process_bio
  dm_make_request
  generic_make_request
  blk_throtl_dispatch_work_fn
  process_one_work
  worker_thread
  kthread
  ret_from_fork
    --
    kworker/3:1 [21912]
    1
``` 

jbd2堆栈: 

```
  blk_throtl_bio
  generic_make_request
  submit_bio
  submit_bh_wbc
  submit_bh
  journal_submit_commit_record
  jbd2_journal_commit_transaction
  kjournald2
  kthread
  ret_from_fork
    --
    jbd2/dm-1-8 [20359]
    4
``` 

发现: kworker堆栈中, 并不包括 ext4_writepages 的堆栈, 也就是说 ext4_writepages并不计入 cgroup 的限制中.

而jbd2计入cgroup的限制中

# 对MySQL IO的观察手段

使用 blkio.throttle.write_iops_device = 10 的MySQL实例

使用如下命令抓取MySQL的IO: 

```
/usr/share/bcc/tools/stackcount -P -Tdi 1 -p $(pgrep mysqld$) -U -r "(blk_start_request|blk_mq_start_request)"
``` 

不使用 blk_throtl_bio 的原因: bio请求, 在通过blk_throtl_bio检查后, 可能被限流. blk_account_io_start是真正的bio开始点

获取每秒的堆栈个数, 小于10, 部分IO被jbd2占用

# 观察MySQL 的IO 举例

```
11:04:45
  fsync
  os_file_flush_func(int)
  pfs_os_file_flush_func(pfs_os_file_t, char const*, unsigned long)
  fil_flush(unsigned long)
  log_write_flush_to_disk_low()
  log_write_up_to(unsigned long, bool)
  trx_flush_log_if_needed_low(unsigned long)
  trx_flush_log_if_needed(unsigned long, trx_t*)
  trx_commit_in_memory(trx_t*, mtr_t const*, bool)
  trx_commit_low(trx_t*, mtr_t*)
  trx_commit(trx_t*)
  trx_commit_for_mysql(trx_t*)
  dict_stats_save(dict_table_t*, unsigned long const*)
  dict_stats_update(dict_table_t*, dict_stats_upd_option_t)
  dict_stats_process_entry_from_recalc_pool()
  dict_stats_thread
  start_thread
    1

  fsync
  os_file_flush_func(int)
  pfs_os_file_flush_func(pfs_os_file_t, char const*, unsigned long)
  fil_flush(unsigned long)
  log_write_flush_to_disk_low()
  log_write_up_to(unsigned long, bool)
  log_buffer_sync_in_background(bool)
  srv_sync_log_buffer_in_background()
  srv_master_do_active_tasks()
  srv_master_thread
  start_thread
    1

  fsync
  os_file_flush_func(int)
  pfs_os_file_flush_func(pfs_os_file_t, char const*, unsigned long)
  fil_flush(unsigned long)
  log_write_flush_to_disk_low()
  log_write_up_to(unsigned long, bool)
  trx_flush_log_if_needed_low(unsigned long)
  trx_flush_log_if_needed(unsigned long, trx_t*)
  trx_commit_in_memory(trx_t*, mtr_t const*, bool)
  trx_commit_low(trx_t*, mtr_t*)
  trx_commit(trx_t*)
  trx_commit_for_mysql(trx_t*)
  dict_stats_exec_sql(pars_info_t*, char const*, trx_t*)
  dict_stats_save(dict_table_t*, unsigned long const*)
  dict_stats_update(dict_table_t*, dict_stats_upd_option_t)
  dict_stats_process_entry_from_recalc_pool()
  dict_stats_thread
  start_thread
    2

11:04:47
  fsync
  os_file_flush_func(int)
  pfs_os_file_flush_func(pfs_os_file_t, char const*, unsigned long)
  fil_flush(unsigned long)
  log_write_flush_to_disk_low()
  log_write_up_to(unsigned long, bool)
  log_buffer_sync_in_background(bool)
  srv_sync_log_buffer_in_background()
  srv_master_do_active_tasks()
  srv_master_thread
  start_thread
    1

11:04:48
  fsync
  os_file_flush_func(int)
  pfs_os_file_flush_func(pfs_os_file_t, char const*, unsigned long)
  fil_flush(unsigned long)
  fil_space_extend(fil_space_t*, unsigned long)
  fsp_try_extend_data_file(fil_space_t*, unsigned char*, mtr_t*)
  fsp_reserve_free_extents(unsigned long*, unsigned long, unsigned long, fsp_reserve_t, mtr_t*, unsigned long)
  btr_cur_pessimistic_insert(unsigned long, btr_cur_t*, unsigned long**, mem_block_info_t**, dtuple_t*, unsigned char**, big_rec_t**, unsigned long, que_thr_t*, mtr_t*)
  row_ins_clust_index_entry_low(unsigned long, unsigned long, dict_index_t*, unsigned long, dtuple_t*, unsigned long, que_thr_t*, bool)
  row_ins_clust_index_entry(dict_index_t*, dtuple_t*, que_thr_t*, unsigned long, bool)
  row_ins_index_entry(dict_index_t*, dtuple_t*, que_thr_t*)
  row_ins_index_entry_step(ins_node_t*, que_thr_t*)
  row_ins(ins_node_t*, que_thr_t*)
  row_ins_step(que_thr_t*)
  row_insert_for_mysql_using_ins_graph(unsigned char const*, row_prebuilt_t*)
  row_insert_for_mysql(unsigned char const*, row_prebuilt_t*)
  ha_innobase::write_row(unsigned char*)
  handler::ha_write_row(unsigned char*)
  Write_rows_log_event::write_row(Relay_log_info const*, bool)
  Write_rows_log_event::do_exec_row(Relay_log_info const*)
  Rows_log_event::do_apply_row(Relay_log_info const*)
  Rows_log_event::do_apply_event(Relay_log_info const*)
  Log_event::apply_event(Relay_log_info*)
  apply_event_and_update_pos(Log_event**, THD*, Relay_log_info*)
  exec_relay_log_event(THD*, Relay_log_info*)
  handle_slave_sql
  pfs_spawn_thread
  start_thread
    1

  fsync
  os_file_flush_func(int)
  pfs_os_file_flush_func(pfs_os_file_t, char const*, unsigned long)
  fil_flush(unsigned long)
  log_write_flush_to_disk_low()
  log_write_up_to(unsigned long, bool)
  log_buffer_sync_in_background(bool)
  srv_sync_log_buffer_in_background()
  srv_master_do_active_tasks()
  srv_master_thread
  start_thread
    3

``` 

# 分析百胜问题的尝试

问题: slave比master慢, IO顶到了cgroup限制

首先执行 biotop: 

样例输出: 

![image2022-7-8 19:16:45.png](/assets/01KJBYKCX4BME84E34QYAKJMAZ/image2022-7-8%2019%3A16%3A45.png)

确定是否是mysqld IO高, 然后通过 上述stackcount方法观察IO的来源
