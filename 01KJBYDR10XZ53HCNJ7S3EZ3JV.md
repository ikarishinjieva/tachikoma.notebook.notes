---
title: 20210721 - p_s 内存统计, 线程统计值和全局统计值的关系
confluence_page_id: 1343524
created_at: 2021-07-21T13:29:31+00:00
updated_at: 2021-07-22T15:09:32+00:00
---

# 实验测试

MySQL 版本: 8.0.25

开启p_s的内存监控: 

```
CALL sys.ps_setup_reset_to_default(FALSE);
CALL sys.ps_setup_enable_instrument('memory/');
``` 

对某表进行自我翻倍: 

```
insert into a select a + (select count(1) from a) from a;
``` 

观察全局内存状况: 

```
select * from memory_summary_global_by_event_name where event_name like '%physical_disk%';
``` 

观察局部内存状况: 

```
select * from memory_summary_by_thread_by_event_name where event_name like '%physical_disk%' and COUNT_ALLOC > 0;
``` 

调试发现, 当经过代码: 

```
#0  PFS_memory_safe_stat::count_alloc (this=0x7fe5402dd9f8, size=1048576, delta=0x7fe5380fddc0) at ../../../mysql-8.0.25/storage/perfschema/pfs_stat.cc:75
#1  0x0000000004f5478b in pfs_memory_alloc_vc (key=410, size=1048576, owner=0x7fe5380fde40) at ../../../mysql-8.0.25/storage/perfschema/pfs.cc:7543
#2  0x000000000500f782 in temptable::Block_PSI_track_physical_disk_allocation (size=1048576) at ../../../mysql-8.0.25/storage/temptable/src/block.cc:129
#3  0x000000000501b5d3 in temptable::allocate_from (src=temptable::Source::MMAP_FILE, size=1048576) at ../../../mysql-8.0.25/storage/temptable/include/temptable/block.h:283
#4  0x000000000501b619 in temptable::Block::Block (this=0x7fe5380fdee8, size=1048576, memory_source=temptable::Source::MMAP_FILE) at ../../../mysql-8.0.25/storage/temptable/include/temptable/block.h:307
#5  0x000000000501c7c0 in temptable::Allocator<unsigned char, temptable::Allocation_scheme<temptable::Exponential_policy, temptable::Prefer_RAM_over_MMAP_policy> >::allocate (this=0x7fe48800f880, n_elements=65536) at ../../../mysql-8.0.25/storage/temptable/include/temptable/allocator.h:491
#6  0x000000000501dcea in temptable::Storage::allocate_back (this=0x7fe48800f898) at ../../../mysql-8.0.25/storage/temptable/include/temptable/storage.h:530
#7  0x000000000501cd83 in temptable::Table::insert (this=0x7fe48800f880, mysql_row=0x7fe488a7af08 "\375\367\237\020") at ../../../mysql-8.0.25/storage/temptable/src/table.cc:115
#8  0x0000000005011723 in temptable::Handler::write_row (this=0x7fe488a72ac8, mysql_row=0x7fe488a7af08 "\375\367\237\020") at ../../../mysql-8.0.25/storage/temptable/src/handler.cc:651
#9  0x000000000352af94 in handler::ha_write_row (this=0x7fe488a72ac8, buf=0x7fe488a7af08 "\375\367\237\020") at ../../mysql-8.0.25/sql/handler.cc:7832
#10 0x00000000038484e2 in MaterializeIterator::MaterializeQueryBlock (this=0x7fe488a4c900, query_block=..., stored_rows=0x7fe5380fe238) at ../../mysql-8.0.25/sql/composite_iterators.cc:838
#11 0x0000000003847b32 in MaterializeIterator::Init (this=0x7fe488a4c900) at ../../mysql-8.0.25/sql/composite_iterators.cc:629
#12 0x00000000032d2283 in Query_expression::ExecuteIteratorQuery (this=0x7fe488b06a68, thd=0x7fe488000f20) at ../../mysql-8.0.25/sql/sql_union.cc:1223
#13 0x00000000032d2611 in Query_expression::execute (this=0x7fe488b06a68, thd=0x7fe488000f20) at ../../mysql-8.0.25/sql/sql_union.cc:1283
#14 0x000000000322be9c in Sql_cmd_dml::execute_inner (this=0x7fe488a709e8, thd=0x7fe488000f20) at ../../mysql-8.0.25/sql/sql_select.cc:791
#15 0x000000000322b418 in Sql_cmd_dml::execute (this=0x7fe488a709e8, thd=0x7fe488000f20) at ../../mysql-8.0.25/sql/sql_select.cc:575
#16 0x00000000031af265 in mysql_execute_command (thd=0x7fe488000f20, first_level=true) at ../../mysql-8.0.25/sql/sql_parse.cc:3426
#17 0x00000000031b43b0 in dispatch_sql_command (thd=0x7fe488000f20, parser_state=0x7fe5380ffb70) at ../../mysql-8.0.25/sql/sql_parse.cc:5000
#18 0x00000000031aa9d2 in dispatch_command (thd=0x7fe488000f20, com_data=0x7fe538100c20, command=COM_QUERY) at ../../mysql-8.0.25/sql/sql_parse.cc:1841
#19 0x00000000031a8e0c in do_command (thd=0x7fe488000f20) at ../../mysql-8.0.25/sql/sql_parse.cc:1320
#20 0x000000000337b4a7 in handle_connection (arg=0x950af50) at ../../mysql-8.0.25/sql/conn_handler/connection_handler_per_thread.cc:301
#21 0x0000000004f499d3 in pfs_spawn_thread (arg=0xb0473c0) at ../../../mysql-8.0.25/storage/perfschema/pfs.cc:2898
#22 0x00007fe549caf6db in start_thread (arg=0x7fe538101700) at pthread_create.c:463
#23 0x00007fe547c7571f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

![image2021-7-21 21:29:1.png](/assets/01KJBYDR10XZ53HCNJ7S3EZ3JV/image2021-7-21%2021%3A29%3A1.png)

对m_alloc_count++操作时, 全局统计和局部统计同时增加

# p_s中memory的统计机制

调查 table_mems_global_by_event_name 的相关代码

重点代码: 

![image2021-7-22 20:30:32.png](/assets/01KJBYDR10XZ53HCNJ7S3EZ3JV/image2021-7-22%2020%3A30%3A32.png)

访问 table_mems_global_by_event_name时, 会使用了iterator遍历并计算结果

非全局的instrument, 会遍历 hosts/accounts/threads 三项, 还会遍历 global 一项 (逻辑藏在子函数内)

  1. p_s 一共有6个记录槽
  2. 从各入口记录内存, 会将内存使用向不同的记录槽记录

     1. PFS_thread::carry_memory_stat_alloc_delta 向thread槽入口记录内存  

        1. 计入相关account槽
        2. 如果没有相关account, 重复计入user槽和host槽
        3. 如果没有相关account, 没有相关user和host, 计入global槽
     2. PFS_user::carry_memory_stat_alloc_delta 向user入口记录内存
        1. 计入相关user槽
     3. PFS_host::carry_memory_stat_alloc_delta 向host入口记录内存
        1. 计入相关host槽
        2. 将剩余计入global槽
     4. PFS_account::carry_memory_stat_alloc_delta 向account入口记录内存
        1. 计入相关account槽
        2. 将 剩余 重复计入user槽和host槽
        3. 如果没有相关user和host, 计入global槽
  3. 访问p_s的表, 会从各个槽中汇总, 如下表: 

|  | global槽 | hosts槽 | users槽 | accounts槽 | threads槽 | THDs槽 |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| mems_global_by_event_name | 全局instrument | x |  |  |  |  |  |
| 非全局instrument | x | x |  | x | x |  |  |
| mems_by_user_by_event_name |  |  |  | 相关user | account.user = 相关user | thread.account.user = 相关user |  |
| mems_by_thread_by_event_name |  |  |  |  |  | x |  |
| mems_by_host_by_event_name |  |  | 相关host |  | account.host = 相关host | thread.account.host = 相关host |  |
| mems_by_account_by_event_name |  |  |  |  | 相关account | thread.account = 相关account |  |
