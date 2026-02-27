---
title: 20221123 - 光大 AST is too big 报错后续
confluence_page_id: 2130229
created_at: 2022-11-23T06:02:56+00:00
updated_at: 2022-11-23T06:02:56+00:00
---

# 前继

[20221020 - 光大clickhouse Too many parts报错]

# 现象

客户进行了参数变更: 

```
alter table table_test4 modify setting prefer_fetch_merged_part_size_threshold=1;
alter table table_test4 modify setting prefer_fetch_merged_part_time_threshold=1;
``` 

但并未生效, 在日志中未找到 Prefer相关日志: 

```
2022.10.22 03:55:26.296103 [ 10 ] {} <Debug> default.table_test4: Prefer to fetch 201901_307_328_1_409 from replica 60-163
``` 

system.mutations表中, is_done=0的记录数也没有变化

可以认为 参数变更 并未生效

# 诊断

检查日志, 发现问题日志: 

```
2022.11.22 17:54:12.934025 [ 16 ] {} <Trace> default.CK_BLACK_IP_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_72710_73034_13_101531 to 4f65688afb3794b6d51e09142c1191bb_72710_73034_13_105132
2022.11.22 17:54:12.936370 [ 16 ] {} <Debug> DiskLocal: Reserving 15.04 MiB on disk `default`, having unreserved 19.89 TiB.
2022.11.22 17:54:12.937922 [ 17 ] {} <Debug> default.CK_BLACK_IP_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_73035_73207_12_105132 because source parts size (7.49 MiB) is greater than the current maximum (0.00 B).
2022.11.22 17:54:12.937963 [ 17 ] {} <Debug> default.CK_BLACK_IP_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_73208_73432_13_105132 because source parts size (9.57 MiB) is greater than the current maximum (0.00 B).
2022.11.22 17:54:12.937979 [ 17 ] {} <Debug> default.CK_BLACK_IP_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_73433_73587_12_105132 because source parts size (6.76 MiB) is greater than the current maximum (0.00 B).
2022.11.22 17:54:12.937996 [ 17 ] {} <Debug> default.CK_BLACK_IP_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_73588_73588_0_105132 because source parts size (73.38 KiB) is greater than the current maximum (0.00 B).
``` 

对 "Not executing log entry MUTATE_PART" 日志的分析: 

代码位置: ReplicatedMergeTreeQueue::shouldExecuteLogEntry

![image2022-11-23 13:12:23.png](/assets/01KJBYVTTVNYTSRBX5JBT2MJKP/image2022-11-23%2013%3A12%3A23.png)

当出现 "Not executing log entry MUTATE_PART" 日志时, shouldExecuteLogEntry返回false, 意为 CK 认为当前没有资源来 回放mutation.

因此也不会触发 prefer_fetch_merged_part_size_threshold/prefer_fetch_merged_part_time_threshold 的逻辑 (在回放mutation时, 让CK直接获取数据, 而并非执行回放)

相关代码路径: 

```
StorageReplicatedMergeTree::queueTask
	- selectEntryToProcess (查找需要执行的Entry)
		- shouldExecuteLogEntry (此处返回false, 该entry代表的mutation不会执行)
	- executeLogEntry
		- tryExecutePartMutation
			- Prefer参数的逻辑判断在此处
``` 

根据日志, max_source_parts_size值为0, 需要调查 merger_mutator.getMaxSourcePartSizeForMutation() 逻辑

![image2022-11-23 13:28:6.png](/assets/01KJBYVTTVNYTSRBX5JBT2MJKP/image2022-11-23%2013%3A28%3A6.png)

当返回值为0时, 意味着:

```
后台线程池中空闲的线程数 < 配置 number_of_free_entries_in_pool_to_execute_mutation (默认值10) 
 
后台线程池中空闲的线程数 = 后台线程池线程总数 (background_pool_size, 默认值16) - 当前繁忙的线程数 (busy_threads_in_pool)
``` 

判断: CK后台线程池中, 有多于6个后台线程在变更时 正在繁忙 (时间段: 2022.11.22 17:42:55 - 2022.11.22 19:46:59以后, 日志到2022.11.22 19:46:59为止)

如何检查后台繁忙线程数 busy_threads_in_pool : 

```
SELECT * FROM system.metrics where metric = 'BackgroundPoolTask'
``` 

建议: 

  1. 扩大 background_pool_size 参数 (在问题CK版本上不适用, 无法动态修改. 动态修改功能在22.5后才能使用)
  2. 等待后台线程不繁忙时, 再进行变更
  3. 缩小 number_of_free_entries_in_pool_to_execute_mutation 参数

```
alter table table_test4 modify setting number_of_free_entries_in_pool_to_execute_mutation=x; (取决于繁忙线程数busy_threads_in_pool)
```
