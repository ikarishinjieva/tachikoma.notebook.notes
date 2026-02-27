---
title: 20221208 - 光大CK DELETE数据失败
confluence_page_id: 2130351
created_at: 2022-12-08T03:37:20+00:00
updated_at: 2022-12-08T03:45:41+00:00
---

CK版本: 20.1.8.41

失败现象: 未知?? 

架构: 26/27两台CK构成复制关系

检查27日志: 

```
2022.11.22 19:08:53.438869 [ 151 ] {} <Debug> DDLWorker: Processing tasks
2022.11.22 19:08:53.438895 [ 147 ] {} <Debug> DDLWorker: Cleaning queue
 
//开始处理DDL
2022.11.22 19:08:53.439675 [ 151 ] {} <Debug> DDLWorker: Processing task query-0000000062 (ALTER TABLE default.CK_BLACK_FILE_LOCAL ON CLUSTER cluster_ck DELETE WHERE (SOURCE = 'ALY') AND (CREATE_TIME <= '2022-08-01 00:00:00') )
2022.11.22 19:08:53.440483 [ 151 ] {} <Debug> DDLWorker: Executing query: ALTER TABLE default.CK_BLACK_FILE_LOCAL DELETE WHERE (SOURCE = 'ALY') AND (CREATE_TIME <= '2022-08-01 00:00:00')
 
//DDL由另外一台CK(leader)进行处理
2022.11.22 19:08:53.823916 [ 151 ] {} <Debug> DDLWorker: Task query-0000000062 has already been executed by leader replica (10%2E214%2E113%2E26:24902) of the same shard.
 
//等待 其他DDL 发生
2022.11.22 19:08:53.825199 [ 151 ] {} <Debug> DDLWorker: Waiting a watch
``` 

检查26日志: 

```
//客户端连到26, 下DDL
2022.11.22 19:08:53.402679 [ 641 ] {8df92057-0893-4247-bf1c-d0be59a5819d} <Debug> executeQuery: (from 10.214.113.26:43956, user: stix) ALTER TABLE CK_BLACK_FILE_LOCAL ON CLUSTER cluster_ck DELETE WHERE (SOURCE = 'ALY') AND (CREATE_TIME
<= '2022-08-01 00:00:00')
2022.11.22 19:08:53.438698 [ 35 ] {} <Debug> DDLWorker: Processing tasks
2022.11.22 19:08:53.438716 [ 2 ] {} <Debug> DDLWorker: Cleaning queue
2022.11.22 19:08:53.439207 [ 641 ] {8df92057-0893-4247-bf1c-d0be59a5819d} <Debug> executeQuery: Query pipeline:
DDLQueryStatusInputStream

//开始处理DDL
2022.11.22 19:08:53.439450 [ 35 ] {} <Debug> DDLWorker: Processing task query-0000000062 (ALTER TABLE default.CK_BLACK_FILE_LOCAL ON CLUSTER cluster_ck DELETE WHERE (SOURCE = 'ALY') AND (CREATE_TIME <= '2022-08-01 00:00:00') )
2022.11.22 19:08:53.440187 [ 35 ] {} <Debug> DDLWorker: Executing query: ALTER TABLE default.CK_BLACK_FILE_LOCAL DELETE WHERE (SOURCE = 'ALY') AND (CREATE_TIME <= '2022-08-01 00:00:00')
 
2022.11.22 19:08:53.444936 [ 35 ] {b09b949f-7d1f-4113-82d0-d6a48b6b58bf} <Debug> executeQuery: (from 0.0.0.0:0, user: ) /* ddl_entry=query-0000000062 */ ALTER TABLE default.CK_BLACK_FILE_LOCAL DELETE WHERE (SOURCE = 'ALY') AND (CREATE_TIME <= '2022-08-01 00:00:00')
2022.11.22 19:08:53.445170 [ 35 ] {b09b949f-7d1f-4113-82d0-d6a48b6b58bf} <Debug> InterpreterSelectQuery: MergeTreeWhereOptimizer: condition "NOT ((SOURCE = 'ALY') AND (CREATE_TIME <= '2022-08-01 00:00:00'))" moved to PREWHERE
 
//DDL处理: 创建mutation 0000064613
2022.11.22 19:08:53.447550 [ 35 ] {b09b949f-7d1f-4113-82d0-d6a48b6b58bf} <Trace> default.CK_BLACK_FILE_LOCAL: Created mutation with ID 0000064613
 
//执行完成
2022.11.22 19:08:53.449859 [ 35 ] {b09b949f-7d1f-4113-82d0-d6a48b6b58bf} <Debug> DDLWorker: Executed query: ALTER TABLE default.CK_BLACK_FILE_LOCAL DELETE WHERE (SOURCE = 'ALY') AND (CREATE_TIME <= '2022-08-01 00:00:00')
2022.11.22 19:08:53.451660 [ 35 ] {b09b949f-7d1f-4113-82d0-d6a48b6b58bf} <Debug> DDLWorker: Waiting a watch
``` 

DDL已经正确创建了mutation

猜测用户CK配置中, mutations_sync使用默认值0, 所以不会等待 本CK 完成mutations回放, DDL即返回完成

在26日志中, 找到mutation回放日志: 

```
//读取到变更的mutation
2022.11.22 19:08:53.461479 [ 73 ] {} <Information> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Loading 1 mutation entries: 0000064613 - 0000064613
 
...
 
 
//无法对CK_BLACK_FILE_LOCAL进行mutation回放
2022.11.22 19:08:53.552547 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89825_89880_4_107204 because source parts size (19.02 MiB) is greater than the current maximum (0.00 B).
2022.11.22 19:08:53.552559 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89636_89643_2_107204 because source parts size (2.67 MiB) is greater than the current maximum (0.00 B).
2022.11.22 19:08:53.552570 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89646_89678_3_107204 because source parts size (10.94 MiB) is greater than the current maximum (0.00 B).
2022.11.22 19:08:53.552581 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89891_89962_4_107204 because source parts size (24.60 MiB) is greater than the current maximum (0.00 B).
2022.11.22 19:08:53.552594 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89965_89965_0_107204 because source parts size (4.10 KiB) is greater than the current maximum (0.00 B).
2022.11.22 19:08:53.552605 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89687_89688_1_107204 because source parts size (337.56 KiB) is greater than the current maximum (0.00 B).
2022.11.22 19:08:53.552616 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89691_89752_4_107204 because source parts size (21.05 MiB) is greater than the current maximum (0.00 B).
2022.11.22 19:08:53.552627 [ 11 ] {} <Debug> default.CK_BLACK_FILE_LOCAL (ReplicatedMergeTreeQueue): Not executing log entry MUTATE_PART for part 4f65688afb3794b6d51e09142c1191bb_89761_89815_4_107204 because source parts size (18.31 MiB) is greater than the current maximum (0.00 B).
``` 

无法回放的原因如之前故障报告所示, 后台繁忙线程数过高, 导致CK认为无资源进行mutation回放
