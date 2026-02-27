---
title: 20210501 - Clickhouse集群 Distributed Engine 学习
confluence_page_id: 754051
created_at: 2021-05-01T15:38:44+00:00
updated_at: 2021-05-08T17:21:06+00:00
---

# 文档1学习

<https://clickhouse.tech/docs/en/engines/table-engines/special/distributed/>

  - Distributed engine 不实际存储数据, 只是逻辑上的表
  - 读请求会发送到所有server. 如果是聚合请求, 各server会分担计算, 产生结果的中间形式, 再聚合产生结果
    - 测试: 
      - select * from all_tt limit 1 仅发往当前节点
      - select * from all_tt limit 2 发往两个节点
      - select * from all_tt order by a desc limit 1 发往两个节点
    - SQL变换形式 ??
  - load_balancing
    - 对于同一个shard的多个replica, 根据load_balancing配置确定请求分发的方式
    - distributed_replica_max_ignored_errors ??
  - 状态查看: system.clusters
  - 集群配置写在config文件中, 重载配置不需要重启
  - remote函数, 可以用于连接远程表: 
    - 形式: remote('addresses_expr', db.table[, 'user'[, 'password'], sharding_key])

    - 举例: SELECT * FROM remote('127.0.0.1', db.remote_engine_table) LIMIT 3;
  - 如何插入数据
    - 可以直接向物理表插入数据 (Clickhouse允许使用任意分片算法)
    - 可以向Distributed逻辑表插入数据, 但必须包括分片键
  - weight
    - 不同shard的数据量权重
  - internal_replication
    - 如果物理表使用了Replicated表, 可设置internal_replication为true, 对shard的写请求, 会发往第一个健康的replica
    - 如果设置internal_replication为false, 写请求会发往所有replica, 但不能保证数据的一致性
  - 分片函数可以配成任何参数, 包括rand()
  - SELECT的处理  

    - 增加新分片时, 可以不迁移数据, 只需要增大权重, 让更多的数据写入新分片
    - 如果使用IN or JOIN, 并且命中分片键, 可以使用local IN/JOIN替换GLOBAL IN/JOIN ??
  - 写请求的处理
    - 数据先写入本地文件, 再异步发往远程服务器
      - distributed_directory_monitor_sleep_time_ms
      - distributed_directory_monitor_max_sleep_time_ms
      - distributed_directory_monitor_batch_inserts
      - background_distributed_schedule_pool_size
    - 重启会导致INSERT数据丢失
    - 如果INSERT数据损坏, 会归入broken文件夹
  - 虚拟列: _shard_num

# 如何观察分区表的SQL下发

查看 system.query_log: 当前节点并不记录下发的SQL, 远端节点会记录下发的SQL (但记录的SQL与原SQL一致, 并不是实际的SQL逻辑)

查看 trace log, 将日志级别调整到DEBUG. 查看日志, 举例如下: 

```
当前节点日志: 
 
2021.05.04 04:57:35.824237 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Debug> executeQuery: (from [::ffff:127.0.0.1]:54206, using production parser) select * from all_tt;
2021.05.04 04:57:35.827041 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON default.all_tt
2021.05.04 04:57:35.828712 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON default.all_tt
2021.05.04 04:57:35.830693 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON db2.tt
2021.05.04 04:57:35.831452 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Debug> db2.tt (f6d7ccd6-1193-43db-b554-c6b74fbbbb0e) (SelectExecutor): Key condition: unknown
2021.05.04 04:57:35.831680 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Debug> db2.tt (f6d7ccd6-1193-43db-b554-c6b74fbbbb0e) (SelectExecutor): Selected 0/0 parts by partition key, 0 parts by primary key, 0/0 marks by primary key, 0 marks to read from 0 ranges
2021.05.04 04:57:35.832227 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Trace> InterpreterSelectQuery: FetchColumns -> WithMergeableState
2021.05.04 04:57:35.833474 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Trace> InterpreterSelectQuery: WithMergeableState -> Complete
2021.05.04 04:57:35.847223 [ 24357 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Trace> PipelineExecutor: Thread finished. Total time: 0.009274723 sec. Execution time: 0.004695031 sec. Processing time: 0.000287376 sec. Wait time: 0.004292316 sec.
2021.05.04 04:57:35.847223 [ 24364 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Trace> PipelineExecutor: Thread finished. Total time: 0.009264212 sec. Execution time: 0.001095238 sec. Processing time: 0.000142836 sec. Wait time: 0.008026138 sec.
2021.05.04 04:57:35.848467 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Information> executeQuery: Read 1 rows, 8.00 B in 0.023852382 sec., 41 rows/sec., 335.40 B/sec.
2021.05.04 04:57:35.848678 [ 24309 ] {9dc439aa-225c-4230-ae7b-3948b423fbdf} <Debug> MemoryTracker: Peak memory usage (for query): 0.00 B.
 
远端节点日志: 
 
2021.05.04 04:57:35.832638 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Debug> executeQuery: (from [::ffff:10.186.62.73]:35596, initial_query_id: 9dc439aa-225c-4230-ae7b-3948b423fbdf, using production parser) SELECT tt.a, tt.b FROM db2.tt
2021.05.04 04:57:35.834020 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON db2.tt
2021.05.04 04:57:35.834333 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Debug> db2.tt (65f8a6f1-8380-4c86-94d3-5ee1592c220d) (SelectExecutor): Key condition: unknown
2021.05.04 04:57:35.834442 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Debug> db2.tt (65f8a6f1-8380-4c86-94d3-5ee1592c220d) (SelectExecutor): Selected 1/1 parts by partition key, 1 parts by primary key, 1/2 marks by primary key, 1 marks to read from 1 ranges
2021.05.04 04:57:35.834594 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Trace> db2.tt (65f8a6f1-8380-4c86-94d3-5ee1592c220d) (SelectExecutor): Reading approx. 1 rows with 1 streams
2021.05.04 04:57:35.834767 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Trace> InterpreterSelectQuery: FetchColumns -> WithMergeableState
2021.05.04 04:57:35.836144 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Information> executeQuery: Read 1 rows, 8.00 B in 0.003245428 sec., 308 rows/sec., 2.41 KiB/sec.
2021.05.04 04:57:35.836227 [ 7505 ] {66e6c482-ec67-4a32-8c31-025f834ba2a9} <Debug> MemoryTracker: Peak memory usage (for query): 0.00 B.
``` 

分析: 

  - 主SQL ID为 c984d1af-fc9b-47ec-9c49-edf703203cb3
  - 看不到 主SQL 向本节点下发的SQL内容
  - 向远端节点下发的SQL ID为 932f1be1-0fcb-4795-bb80-6f925f8063b1  

    - 但记录SQL始终于原SQL一致, 而不是实际下发到节点上的SQL逻辑 (使用group by等聚合函数, 可以测试其逻辑并不正确), 需要其他方法观察远端节点的SQL是什么
  - 与主SQL无法建立明确关联, 但通过 --send_logs_level=trace 可以讲两者关联起来, 如下:   

```
[ubuntu] 2021.05.04 05:04:54.795014 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Debug> executeQuery: (from [::ffff:127.0.0.1]:54206, using production parser) select * from all_tt;
[ubuntu] 2021.05.04 05:04:54.797059 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON default.all_tt
[ubuntu] 2021.05.04 05:04:54.798579 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON default.all_tt
[ubuntu] 2021.05.04 05:04:54.800129 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON db2.tt
[ubuntu] 2021.05.04 05:04:54.800691 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Debug> db2.tt (f6d7ccd6-1193-43db-b554-c6b74fbbbb0e) (SelectExecutor): Key condition: unknown
[ubuntu] 2021.05.04 05:04:54.800837 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Debug> db2.tt (f6d7ccd6-1193-43db-b554-c6b74fbbbb0e) (SelectExecutor): Selected 0/0 parts by partition key, 0 parts by primary key, 0/0 marks by primary key, 0 marks to read from 0 ranges
[ubuntu] 2021.05.04 05:04:54.801243 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> InterpreterSelectQuery: FetchColumns -> WithMergeableState
[ubuntu] 2021.05.04 05:04:54.802396 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> InterpreterSelectQuery: WithMergeableState -> Complete
[ubuntu] 2021.05.04 05:04:55.211746 [ 24363 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Warning> HedgedConnectionsFactory: Connection failed at try №1, reason: Code: 32, e.displayText() = DB::Exception: Attempt to read after eof (version 21.4.4.1)
[ubuntu] 2021.05.04 05:04:55.212149 [ 24363 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> Connection (10.186.65.37:9000): Connecting. Database: (not specified). User: default
[ubuntu] 2021.05.04 05:04:55.214098 [ 24363 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> Connection (10.186.65.37:9000): Connected to ClickHouse server version 21.4.4.
[ubuntu-2] 2021.05.04 05:04:55.207193 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Debug> executeQuery: (from [::ffff:10.186.62.73]:44082, initial_query_id: 06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327, using production parser) SELECT tt.a, tt.b FROM db2.tt
[ubuntu-2] 2021.05.04 05:04:55.207674 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Trace> ContextAccess (default): Access granted: SELECT(a, b) ON db2.tt
[ubuntu-2] 2021.05.04 05:04:55.207760 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Debug> db2.tt (65f8a6f1-8380-4c86-94d3-5ee1592c220d) (SelectExecutor): Key condition: unknown
[ubuntu-2] 2021.05.04 05:04:55.207848 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Debug> db2.tt (65f8a6f1-8380-4c86-94d3-5ee1592c220d) (SelectExecutor): Selected 1/1 parts by partition key, 1 parts by primary key, 1/2 marks by primary key, 1 marks to read from 1 ranges
[ubuntu-2] 2021.05.04 05:04:55.207992 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Trace> db2.tt (65f8a6f1-8380-4c86-94d3-5ee1592c220d) (SelectExecutor): Reading approx. 1 rows with 1 streams
[ubuntu-2] 2021.05.04 05:04:55.208103 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Trace> InterpreterSelectQuery: FetchColumns -> WithMergeableState
┌─a─┬─b─┐
│ 1 │ 2 │
└───┴───┘
[ubuntu-2] 2021.05.04 05:04:55.209114 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Information> executeQuery: Read 1 rows, 8.00 B in 0.00176319 sec., 567 rows/sec., 4.43 KiB/sec.
[ubuntu-2] 2021.05.04 05:04:55.209182 [ 9863 ] {29522877-9c5b-41d6-992d-453af6d4a591} <Debug> MemoryTracker: Peak memory usage (for query): 0.00 B.
[ubuntu] 2021.05.04 05:04:55.221018 [ 24363 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> PipelineExecutor: Thread finished. Total time: 0.414762337 sec. Execution time: 0.411106257 sec. Processing time: 0.000255202 sec. Wait time: 0.003400878 sec.
[ubuntu] 2021.05.04 05:04:55.221001 [ 24357 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Trace> PipelineExecutor: Thread finished. Total time: 0.414790337 sec. Execution time: 0.002150679 sec. Processing time: 0.00016448 sec. Wait time: 0.412475178 sec.
[ubuntu] 2021.05.04 05:04:55.222284 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Information> executeQuery: Read 1 rows, 8.00 B in 0.426968133 sec., 2 rows/sec., 18.74 B/sec.
[ubuntu] 2021.05.04 05:04:55.222534 [ 24309 ] {06c0e1a6-e0a9-46d6-a9ff-c47cc20ec327} <Debug> MemoryTracker: Peak memory usage (for query): 0.00 B.
```

    - 可通过日志中的hostname来区分主机

# 代码入口点

DB::StorageDistributed::getQueryProcessingStage

```
#0  DB::StorageDistributed::getQueryProcessingStage (this=0x7f79b6435200, context=..., to_stage=DB::QueryProcessingStage::Complete, query_info=...) at ../src/Storages/StorageDistributed.cpp:454
#1  0x000000001a61567a in DB::InterpreterSelectQuery::getSampleBlockImpl (this=0x7f79b8ddc900) at ../src/Interpreters/InterpreterSelectQuery.cpp:577
#2  0x000000001a611768 in DB::InterpreterSelectQuery::InterpreterSelectQuery(std::__1::shared_ptr<DB::IAST> const&, DB::Context const&, std::__1::shared_ptr<DB::IBlockInputStream> const&, std::__1::optional<DB::Pipe>, std::__1::shared_ptr<DB::IStorage> const&, DB::SelectQueryOptions const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::shared_ptr<DB::StorageInMemoryMetadata const> const&)::$_2::operator()(bool) const (
    this=0x7f79bb139c48, try_move_to_prewhere=true) at ../src/Interpreters/InterpreterSelectQuery.cpp:475
#3  0x000000001a60df39 in DB::InterpreterSelectQuery::InterpreterSelectQuery (this=0x7f79b8ddc900, query_ptr_=..., context_=..., input_=..., input_pipe_=..., storage_=..., options_=...,
    required_result_column_names=..., metadata_snapshot_=...) at ../src/Interpreters/InterpreterSelectQuery.cpp:478
#4  0x000000001a60cc74 in DB::InterpreterSelectQuery::InterpreterSelectQuery (this=0x7f79b8ddc900, query_ptr_=..., context_=..., options_=..., required_result_column_names_=...)
    at ../src/Interpreters/InterpreterSelectQuery.cpp:160
#5  0x000000001ab87f39 in std::__1::make_unique<DB::InterpreterSelectQuery, std::__1::shared_ptr<DB::IAST> const&, DB::Context&, DB::SelectQueryOptions&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&> (__args=..., __args=...,
    __args=..., __args=...) at ../contrib/libcxx/include/memory:2068
#6  0x000000001ab86200 in DB::InterpreterSelectWithUnionQuery::buildCurrentChildInterpreter (this=0x7f79b539e440, ast_ptr_=..., current_required_result_column_names=...)
    at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:211
#7  0x000000001ab85b5d in DB::InterpreterSelectWithUnionQuery::InterpreterSelectWithUnionQuery (this=0x7f79b539e440, query_ptr_=..., context_=..., options_=..., required_result_column_names=...)
    at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:133
#8  0x000000001a5a9c94 in std::__1::make_unique<DB::InterpreterSelectWithUnionQuery, std::__1::shared_ptr<DB::IAST>&, DB::Context&, DB::SelectQueryOptions const&> (__args=..., __args=..., __args=...)
    at ../contrib/libcxx/include/memory:2068
#9  0x000000001a5a8338 in DB::InterpreterFactory::get (query=..., context=..., options=...) at ../src/Interpreters/InterpreterFactory.cpp:110
#10 0x000000001ada23ed in DB::executeQueryImpl (begin=0x7f79b1e09550 "select * from all_tt order by a limit 1;", end=0x7f79b1e09578 "", context=..., internal=false,
    stage=DB::QueryProcessingStage::Complete, has_query_tail=false, istr=0x0) at ../src/Interpreters/executeQuery.cpp:520
#11 0x000000001ada0c0a in DB::executeQuery (query=..., context=..., internal=false, stage=DB::QueryProcessingStage::Complete, may_have_embedded_data=true) at ../src/Interpreters/executeQuery.cpp:908
#12 0x000000001b7684b6 in DB::TCPHandler::runImpl (this=0x7f799d031000) at ../src/Server/TCPHandler.cpp:290
#13 0x000000001b775145 in DB::TCPHandler::run (this=0x7f799d031000) at ../src/Server/TCPHandler.cpp:1548
#14 0x000000001f075cc9 in Poco::Net::TCPServerConnection::start (this=0x7f799d031000) at ../contrib/poco/Net/src/TCPServerConnection.cpp:43
#15 0x000000001f0764a9 in Poco::Net::TCPServerDispatcher::run (this=0x7f79b668ce00) at ../contrib/poco/Net/src/TCPServerDispatcher.cpp:113
#16 0x000000001f1a9184 in Poco::PooledThread::run (this=0x7f7a5cd6c480) at ../contrib/poco/Foundation/src/ThreadPool.cpp:199
#17 0x000000001f1a622a in Poco::(anonymous namespace)::RunnableHolder::run (this=0x7f7a5cc598f0) at ../contrib/poco/Foundation/src/Thread.cpp:55
#18 0x000000001f1a50c2 in Poco::ThreadImpl::runnableEntry (pThread=0x7f7a5cd6c4b8) at ../contrib/poco/Foundation/src/Thread_POSIX.cpp:345
#19 0x00007f7a5e00d6db in start_thread (arg=0x7f79bb151700) at pthread_create.c:463
#20 0x00007f7a5dd3671f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 
    
    
    DB::StorageDistributed::read

```
(gdb) bt
#0  DB::StorageDistributed::read (this=0x7f79b6435200, query_plan=..., column_names=..., metadata_snapshot=..., query_info=..., context=..., processed_stage=DB::QueryProcessingStage::WithMergeableState)
    at ../src/Storages/StorageDistributed.cpp:539
#1  0x000000001a61a766 in DB::InterpreterSelectQuery::executeFetchColumns (this=0x7f7977094f00, processing_stage=DB::QueryProcessingStage::WithMergeableState, query_plan=...)
    at ../src/Interpreters/InterpreterSelectQuery.cpp:1729
#2  0x000000001a612ee0 in DB::InterpreterSelectQuery::executeImpl (this=0x7f7977094f00, query_plan=..., prepared_input=..., prepared_pipe=...) at ../src/Interpreters/InterpreterSelectQuery.cpp:965
#3  0x000000001a611f6c in DB::InterpreterSelectQuery::buildQueryPlan (this=0x7f7977094f00, query_plan=...) at ../src/Interpreters/InterpreterSelectQuery.cpp:541
#4  0x000000001ab86bbd in DB::InterpreterSelectWithUnionQuery::buildQueryPlan (this=0x7f7a5d47d740, query_plan=...) at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:243
#5  0x000000001ab8734a in DB::InterpreterSelectWithUnionQuery::execute (this=0x7f7a5d47d740) at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:298
#6  0x000000001ada279b in DB::executeQueryImpl (begin=0x7f79b67322a0 "select * from all_tt order by a limit 1;", end=0x7f79b67322c8 "", context=..., internal=false,
    stage=DB::QueryProcessingStage::Complete, has_query_tail=false, istr=0x0) at ../src/Interpreters/executeQuery.cpp:550
#7  0x000000001ada0c0a in DB::executeQuery (query=..., context=..., internal=false, stage=DB::QueryProcessingStage::Complete, may_have_embedded_data=true) at ../src/Interpreters/executeQuery.cpp:908
#8  0x000000001b7684b6 in DB::TCPHandler::runImpl (this=0x7f799d031000) at ../src/Server/TCPHandler.cpp:290
#9  0x000000001b775145 in DB::TCPHandler::run (this=0x7f799d031000) at ../src/Server/TCPHandler.cpp:1548
#10 0x000000001f075cc9 in Poco::Net::TCPServerConnection::start (this=0x7f799d031000) at ../contrib/poco/Net/src/TCPServerConnection.cpp:43
#11 0x000000001f0764a9 in Poco::Net::TCPServerDispatcher::run (this=0x7f79b668ce00) at ../contrib/poco/Net/src/TCPServerDispatcher.cpp:113
#12 0x000000001f1a9184 in Poco::PooledThread::run (this=0x7f7a5cd6c480) at ../contrib/poco/Foundation/src/ThreadPool.cpp:199
#13 0x000000001f1a622a in Poco::(anonymous namespace)::RunnableHolder::run (this=0x7f7a5cc598f0) at ../contrib/poco/Foundation/src/Thread.cpp:55
#14 0x000000001f1a50c2 in Poco::ThreadImpl::runnableEntry (pThread=0x7f7a5cd6c4b8) at ../contrib/poco/Foundation/src/Thread_POSIX.cpp:345
#15 0x00007f7a5e00d6db in start_thread (arg=0x7f79bb151700) at pthread_create.c:463
#16 0x00007f7a5dd3671f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

# 代码分析 - InterpreterSelectQuery::getSampleBlockImpl

获取结果集的列

  - 调用 storage->getQueryProcessingStage, 获取 from_stage (处理的当前阶段)
  - options.to_stage (处理的下一阶段)
  - 处理阶段定义分为: 

```
    enum Enum
    {
        /// Only read/have been read the columns specified in the query.
        FetchColumns       = 0,
        /// Until the stage where the results of processing on different servers can be combined.
        WithMergeableState = 1,
        /// Completely.
        Complete           = 2,
        /// Until the stage where the aggregate functions were calculated and finalized.
        ///
        /// It is used for auto distributed_group_by_no_merge optimization for distributed engine.
        /// (See comments in StorageDistributed).
        WithMergeableStateAfterAggregation = 3,

        MAX = 4,
    };
```

  - first_stage: bool: 需要从远端节点获取数据, 定义为: 

```
bool first_stage = from_stage < QueryProcessingStage::WithMergeableState
        && options.to_stage >= QueryProcessingStage::WithMergeableState;
```

  - second_stage: bool: 需要在初始节点进行数据汇聚, 定义为: 

```
bool second_stage = from_stage <= QueryProcessingStage::WithMergeableState
        && options.to_stage > QueryProcessingStage::WithMergeableState;
```

  - 将first_stage, second_stage等信息送入 ExpressionAnalysisResult (analysis_result) 分析, 解析出有效信息
  - 根据目标阶段, 从analysis_result选取合适的信息: 
    - FetchColumns阶段: 返回source_header, 结合 analysis_result.prewhere_info
    - WithMergeableState阶段: 提取 analysis_result.before_aggregation->getResultColumns()
    - WithMergeableStateAfterAggregation阶段: 提取analysis_result.before_order_by->getResultColumns()
    - 否则: 提取analysis_result.final_projection->getResultColumns()

# 代码分析 - ExpressionAnalysisResult

  - 只记录一些散点
  - 主要用于构造 ExpressionActionsChain, 形成ActionStep的序列, 即动作序列. 并记录其中主要操作, 比如 before_aggregation, before_having
  - 本函数需要处理SQL的结构, 以下举例: 
    - 处理聚合结构: need_aggregate
      - query_analyzer.appendGroupBy: 将group by的参数, 构造相关action以获取参数
      - query_analyzer.appendAggregateFunctionsArguments: : 将select, having和order by的参数, 构造相关action以获取参数
      - 更新before_aggregation和before_having

# 代码分析杂项

  - getRootActions: 将AST转成actionDAG, rootAction为action的根动作

# 测试

配置新增: 

```
<test_shard>
     <shard>
         <replica>
             <host>10.186.62.73</host>
             <port>9000</port>
         </replica>
     </shard>
     <shard>
         <replica>
             <host>10.186.65.37</host>
             <port>9000</port>
         </replica>
     </shard>
</test_shard>
 
``` 

cluster配置需更新到所有节点, 否则进行集群操作时会报错如下: 

```
Code: 371, e.displayText() = DB::Exception: DDL task query-0000000002 contains current host 10.186.65.37:9000 in cluster test_shard, but there are no such cluster here. (version 21.4.4.30 (official build))
``` 

  
  

安装zookeeper: <https://zookeeper.apache.org/doc/current/zookeeperStarted.html>

配置新增: 

```
<zookeeper>
    <node>
        <host>10.186.62.73</host>
        <port>2181</port>
    </node>
</zookeeper>
``` 

更新zookeeper配置需重启clickhouse ??

创建逻辑表: 

```
CREATE TABLE IF NOT EXISTS all_tt ON CLUSTER test_shard (a Int32, b Int32) ENGINE = Distributed(test_shard, test, tt)
``` 

创建物理表: 

```
CREATE TABLE db2.tt (a Int32, b Int32) ENGINE = MergeTree() ORDER BY a;
``` 

# 单节点与分布式执行计划的不同

单节点执行计划: 

```
localhost :) explain actions=1 select count(1) from db2.tt;

EXPLAIN actions = 1
SELECT count(1)
FROM db2.tt

Query id: 97c4819c-bc0d-4486-a4e2-9be2177a32f6

┌─explain──────────────────────────────────────────────┐
│ Expression ((Projection + Before ORDER BY))          │
│ Actions: INPUT :: 0 -> count() UInt64 : 0            │
│ Positions: 0                                         │
│   MergingAggregated                                  │
│   Keys:                                              │
│   Aggregates:                                        │
│       count()                                        │
│         Function: count() → UInt64                   │
│         Arguments: none                              │
│         Argument positions: none                     │
│     ReadFromPreparedSource (Optimized trivial count) │
└──────────────────────────────────────────────────────┘

11 rows in set. Elapsed: 0.007 sec.
``` 

分布式执行计划: 

```
localhost :) explain actions=1 select count(1) from all_tt;

EXPLAIN actions = 1
SELECT count(1)
FROM all_tt

Query id: c3d90f44-4d7f-4d57-962c-2a7e6e7c1f65

┌─explain─────────────────────────────────────────────────────────────────────┐
│ Expression ((Projection + Before ORDER BY))                                 │
│ Actions: INPUT :: 0 -> count() UInt64 : 0                                   │
│ Positions: 0                                                                │
│   MergingAggregated                                                         │
│   Keys:                                                                     │
│   Aggregates:                                                               │
│       count()                                                               │
│         Function: count() → UInt64                                          │
│         Arguments: none                                                     │
│         Argument positions: none                                            │
│     SettingQuotaAndLimits (Set limits and quota after reading from storage) │
│       Union                                                                 │
│         Expression (Convert block structure for query from local replica)   │
│         Actions: INPUT :: 0 -> count() AggregateFunction(count) : 0         │
│         Positions: 0                                                        │
│           ReadFromPreparedSource (Optimized trivial count)                  │
│         ReadFromPreparedSource (Read from remote replica)                   │
└─────────────────────────────────────────────────────────────────────────────┘

17 rows in set. Elapsed: 0.012 sec.
``` 

差异在于Union操作, 如下处理逻辑里整理了Union操作是来源自DistributedEngine的读数据

  
clickhouse对query的处理逻辑

```
BlockIO InterpreterSelectWithUnionQuery::execute()
{
    BlockIO res;

    QueryPlan query_plan;
    buildQueryPlan(query_plan); 

    auto pipeline = query_plan.buildQueryPipeline(
        QueryPlanOptimizationSettings::fromContext(*context),
        BuildQueryPipelineSettings::fromContext(*context));

    res.pipeline = std::move(*pipeline);
    res.pipeline.addInterpreterContext(context);

    return res;
}
``` 

  - Interpreter 负责从AST生成 QueryPlan
    - Interpreter的构造器, 使用 ExpressionAnalysisResult 用于分析SQL的特性 (内部实现: 生成了ExpressionActionsStep的列表 ??)

```
/** First we compose a chain of actions and remember the necessary steps from it.
        *  Regardless of from_stage and to_stage, we will compose a complete sequence of actions to perform optimization and
        *  throw out unnecessary columns based on the entire query. In unnecessary parts of the query, we will not execute subqueries.
        */
```

      - 其中用 first_stage 和 second_stage (from_stage 和 to_stage), 控制 不生成 非必要的部分
    - buildQueryPlan, 根据SQL的特征, 添加QueryPlanStep (执行计划的步骤)

      - 此处确定从引擎读取数据的步骤, DistributedEngine在此处增加了Union步骤 (ClusterProxy.executeQuery)

```
(gdb) bt
#0  DB::UnionStep::UnionStep (this=0x7f79b66c9e40, input_streams_=..., result_header=..., max_threads_=0) at ../src/Processors/QueryPlan/UnionStep.cpp:9
#1  0x000000001aff8cca in std::__1::make_unique<DB::UnionStep, std::__1::vector<DB::DataStream, std::__1::allocator<DB::DataStream> >, DB::Block&> (__args=..., __args=...)
    at ../contrib/libcxx/include/memory:2068
#2  0x000000001aff84b2 in DB::ClusterProxy::executeQuery (query_plan=..., stream_factory=..., log=0x7f79b20b8c80, query_ast=..., context=..., query_info=...)
    at ../src/Interpreters/ClusterProxy/executeQuery.cpp:160
#3  0x000000001afe0c5f in DB::StorageDistributed::read (this=0x7f79b6435200, query_plan=..., column_names=..., metadata_snapshot=..., query_info=..., context=...,
    processed_stage=DB::QueryProcessingStage::WithMergeableState) at ../src/Storages/StorageDistributed.cpp:567
#4  0x000000001a61a766 in DB::InterpreterSelectQuery::executeFetchColumns (this=0x7f79b6765b00, processing_stage=DB::QueryProcessingStage::WithMergeableState, query_plan=...)
    at ../src/Interpreters/InterpreterSelectQuery.cpp:1729
#5  0x000000001a612ee0 in DB::InterpreterSelectQuery::executeImpl (this=0x7f79b6765b00, query_plan=..., prepared_input=..., prepared_pipe=...) at ../src/Interpreters/InterpreterSelectQuery.cpp:965
#6  0x000000001a611f6c in DB::InterpreterSelectQuery::buildQueryPlan (this=0x7f79b6765b00, query_plan=...) at ../src/Interpreters/InterpreterSelectQuery.cpp:541
#7  0x000000001ab86bbd in DB::InterpreterSelectWithUnionQuery::buildQueryPlan (this=0x7f7a5cdfb200, query_plan=...) at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:243
#8  0x000000001ab8734a in DB::InterpreterSelectWithUnionQuery::execute (this=0x7f7a5cdfb200) at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:298
#9  0x000000001ada279b in DB::executeQueryImpl (begin=0x7f7a5d43a6e0 "select count(1) from all_tt", end=0x7f7a5d43a6fb "", context=..., internal=false, stage=DB::QueryProcessingStage::Complete,
    has_query_tail=false, istr=0x0) at ../src/Interpreters/executeQuery.cpp:550
#10 0x000000001ada0c0a in DB::executeQuery (query=..., context=..., internal=false, stage=DB::QueryProcessingStage::Complete, may_have_embedded_data=true) at ../src/Interpreters/executeQuery.cpp:908
#11 0x000000001b7684b6 in DB::TCPHandler::runImpl (this=0x7f79b5338000) at ../src/Server/TCPHandler.cpp:290
#12 0x000000001b775145 in DB::TCPHandler::run (this=0x7f79b5338000) at ../src/Server/TCPHandler.cpp:1548
#13 0x000000001f075cc9 in Poco::Net::TCPServerConnection::start (this=0x7f79b5338000) at ../contrib/poco/Net/src/TCPServerConnection.cpp:43
#14 0x000000001f0764a9 in Poco::Net::TCPServerDispatcher::run (this=0x7f79b668ce00) at ../contrib/poco/Net/src/TCPServerDispatcher.cpp:113
#15 0x000000001f1a9184 in Poco::PooledThread::run (this=0x7f7a5cd6c480) at ../contrib/poco/Foundation/src/ThreadPool.cpp:199
#16 0x000000001f1a622a in Poco::(anonymous namespace)::RunnableHolder::run (this=0x7f7a5cc598f0) at ../contrib/poco/Foundation/src/Thread.cpp:55
#17 0x000000001f1a50c2 in Poco::ThreadImpl::runnableEntry (pThread=0x7f7a5cd6c4b8) at ../contrib/poco/Foundation/src/Thread_POSIX.cpp:345
#18 0x00007f7a5e00d6db in start_thread (arg=0x7f79bb151700) at pthread_create.c:463
#19 0x00007f7a5dd3671f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

    - 再从QueryPlanStep生成 pipeline, 包装成BlockIO
    - BlockIO实际上是一个数据流, 从中获取数据即可触发pipeline工作
      - 分布式表 发送Query的堆栈: 

```
#0  DB::HedgedConnections::sendQuery (this=0x7f79b8cf1400, timeouts=..., query=..., query_id=..., stage=1, client_info=..., with_pending_data=true) at ../src/Client/HedgedConnections.cpp:133
#1  0x000000001a136b0b in DB::RemoteQueryExecutor::sendQuery (this=0x7f799cf9f420) at ../src/DataStreams/RemoteQueryExecutor.cpp:192
#2  0x000000001bb0ddd2 in DB::RemoteSource::tryGenerate (this=0x7f79b6632618) at ../src/Processors/Sources/RemoteSource.cpp:65
#3  0x000000001b7b7484 in DB::ISource::work (this=0x7f79b6632618) at ../src/Processors/ISource.cpp:53
#4  0x000000001bb156dd in DB::SourceWithProgress::work (this=0x7f79b6632618) at ../src/Processors/Sources/SourceWithProgress.cpp:36
#5  0x000000001b814da9 in DB::executeJob (processor=0x7f79b6632618) at ../src/Processors/Executors/PipelineExecutor.cpp:79
#6  0x000000001b814d1f in DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0::operator()() const (this=0x7f79b67816f0) at ../src/Processors/Executors/PipelineExecutor.cpp:96
#7  0x000000001b814cdd in _ZNSt3__18__invokeIRZN2DB16PipelineExecutor6addJobEPNS1_14ExecutingGraph4NodeEE3$_0JEEEDTclclsr3std3__1E7forwardIT_Efp_Espclsr3std3__1E7forwardIT0_Efp0_EEEOS8_DpOS9_ (__f=...)
    at ../contrib/libcxx/include/type_traits:3676
#8  0x000000001b814cad in std::__1::__invoke_void_return_wrapper<void>::__call<DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0&>(DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0&) (__args=...) at ../contrib/libcxx/include/__functional_base:348
#9  0x000000001b814c85 in std::__1::__function::__default_alloc_func<DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0(void ())>::operator()() (this=0x7f79b67816f0)
    at ../contrib/libcxx/include/functional:1608
#10 0x000000001b814c5d in std::__1::__function::__policy_invoker<void ()>::__call_impl<std::__1::__function::__default_alloc_func<DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0(void ())> >(std::__1::__function::__policy_storage const*) (__buf=0x7f79b67816f0) at ../contrib/libcxx/include/functional:2089
#11 0x0000000011498846 in std::__1::__function::__policy_func<void ()>::operator()() const (this=0x7f79b67816f0) at ../contrib/libcxx/include/functional:2221
#12 0x0000000011497b75 in std::__1::function<void ()>::operator()() const (this=0x7f79b67816f0) at ../contrib/libcxx/include/functional:2560
#13 0x000000001b813727 in DB::PipelineExecutor::executeStepImpl (this=0x7f79b51a0a98, thread_num=1, num_threads=3, yield_flag=0x0) at ../src/Processors/Executors/PipelineExecutor.cpp:585
#14 0x000000001b8140d5 in DB::PipelineExecutor::executeSingleThread (this=0x7f79b51a0a98, thread_num=1, num_threads=3) at ../src/Processors/Executors/PipelineExecutor.cpp:473
#15 0x000000001b815c4f in DB::PipelineExecutor::executeImpl(unsigned long)::$_4::operator()() const (this=0x7f79a9360c40) at ../src/Processors/Executors/PipelineExecutor.cpp:776
#16 0x000000001b815bbd in _ZNSt3__118__invoke_constexprIRZN2DB16PipelineExecutor11executeImplEmE3$_4JEEEDTclclsr3std3__1E7forwardIT_Efp_Espclsr3std3__1E7forwardIT0_Efp0_EEEOS5_DpOS6_ (__f=...)
    at ../contrib/libcxx/include/type_traits:3682
#17 0x000000001b815b81 in _ZNSt3__118__apply_tuple_implIRZN2DB16PipelineExecutor11executeImplEmE3$_4RNS_5tupleIJEEEJEEEDcOT_OT0_NS_15__tuple_indicesIJXspT1_EEEE (__f=..., __t=...)
    at ../contrib/libcxx/include/tuple:1415
#18 0x000000001b815ac2 in std::__1::apply<DB::PipelineExecutor::executeImpl(unsigned long)::$_4(std::__1::tuple<>&)&> (__f=..., __t=...) at ../contrib/libcxx/include/tuple:1424
#19 0x000000001b8159a9 in _ZZN20ThreadFromGlobalPoolC1IZN2DB16PipelineExecutor11executeImplEmE3$_4JEEEOT_DpOT0_ENUlvE_clEv (this=0x7f79b1e89780) at ../src/Common/ThreadPool.h:178
#20 0x000000001b8158ed in _ZNSt3__18__invokeIRZN20ThreadFromGlobalPoolC1IZN2DB16PipelineExecutor11executeImplEmE3$_4JEEEOT_DpOT0_EUlvE_JEEEDTclclsr3std3__1E7forwardIS6_Efp_Espclsr3std3__1E7forwardIS8_Efp0_EEES7_SA_ (__f=...) at ../contrib/libcxx/include/type_traits:3676
#21 0x000000001b8158bd in _ZNSt3__128__invoke_void_return_wrapperIvE6__callIJRZN20ThreadFromGlobalPoolC1IZN2DB16PipelineExecutor11executeImplEmE3$_4JEEEOT_DpOT0_EUlvE_EEEvDpOT_ (__args=...)
    at ../contrib/libcxx/include/__functional_base:348
#22 0x000000001b815895 in _ZNSt3__110__function20__default_alloc_funcIZN20ThreadFromGlobalPoolC1IZN2DB16PipelineExecutor11executeImplEmE3$_4JEEEOT_DpOT0_EUlvE_FvvEEclEv (this=0x7f79b1e89780)
    at ../contrib/libcxx/include/functional:1608
#23 0x000000001b815870 in _ZNSt3__110__function16__policy_invokerIFvvEE11__call_implINS0_20__default_alloc_funcIZN20ThreadFromGlobalPoolC1IZN2DB16PipelineExecutor11executeImplEmE3$_4JEEEOT_DpOT0_EUlvE_S2_EEEEvPKNS0_16__policy_storageE (__buf=0x7f79a9360e28) at ../contrib/libcxx/include/functional:2089
#24 0x0000000011498846 in std::__1::__function::__policy_func<void ()>::operator()() const (this=0x7f79a9360e28) at ../contrib/libcxx/include/functional:2221
#25 0x0000000011497b75 in std::__1::function<void ()>::operator()() const (this=0x7f79a9360e28) at ../contrib/libcxx/include/functional:2560
#26 0x00000000114baad0 in ThreadPoolImpl<std::__1::thread>::worker (this=0x7f7a5cc51100, thread_it=...) at ../src/Common/ThreadPool.cpp:247
#27 0x00000000114c1a34 in void ThreadPoolImpl<std::__1::thread>::scheduleImpl<void>(std::__1::function<void ()>, int, std::__1::optional<unsigned long>)::{lambda()#3}::operator()() const (
    this=0x7f79b66ac628) at ../src/Common/ThreadPool.cpp:124
#28 0x00000000114c19cd in std::__1::__invoke<void ThreadPoolImpl<std::__1::thread>::scheduleImpl<void>(std::__1::function<void ()>, int, std::__1::optional<unsigned long>)::{lambda()#3}> (__f=...)
    at ../contrib/libcxx/include/type_traits:3676
#29 0x00000000114c1925 in std::__1::__thread_execute<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct> >, void ThreadPoolImpl<std::__1::thread>::scheduleImpl<void>(std::__1::function<void ()>, int, std::__1::optional<unsigned long>)::{lambda()#3}>(std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct> >, void ThreadPoolImpl<std::__1::thread>::scheduleImpl<void>(std::__1::function<void ()>, int, std::__1::optional<unsigned long>)::{lambda()#3}>&, std::__1::__tuple_indices<>) (__t=...)
    at ../contrib/libcxx/include/thread:280
#30 0x00000000114c1305 in std::__1::__thread_proxy<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct> >, void ThreadPoolImpl<std::__1::thre---Type <return> to continue, or q <return> to quit---
ad>::scheduleImpl<void>(std::__1::function<void ()>, int, std::__1::optional<unsigned long>)::{lambda()#3}> >(std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct> >, void ThreadPoolImpl<std::__1::thread>::scheduleImpl<void>(std::__1::function<void ()>, int, std::__1::optional<unsigned long>)::{lambda()#3}>) (__vp=0x7f79b66ac620)
    at ../contrib/libcxx/include/thread:291
#31 0x00007f7a5e00d6db in start_thread (arg=0x7f79a936e700) at pthread_create.c:463
#32 0x00007f7a5dd3671f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

    - stage=DB::QueryProcessingStage::WithMergeableState 会被写入网络包, 使得远端节点执行时, 只获取数据后即返回数据, 不进行聚合
    - 分布式表接收Query的堆栈: 

```
(gdb) bt
#0  DB::SelectQueryOptions::SelectQueryOptions (this=0x7f79ba93ee18, stage=DB::QueryProcessingStage::WithMergeableState, depth=0, is_subquery_=false) at ../src/Interpreters/SelectQueryOptions.h:39
#1  0x000000001ada23a9 in DB::executeQueryImpl (begin=0x7f79b20c30a0 "SELECT tt.a, tt.b FROM db2.tt", end=0x7f79b20c30bd "", context=..., internal=false,
    stage=DB::QueryProcessingStage::WithMergeableState, has_query_tail=false, istr=0x0) at ../src/Interpreters/executeQuery.cpp:520
#2  0x000000001ada0c0a in DB::executeQuery (query=..., context=..., internal=false, stage=DB::QueryProcessingStage::WithMergeableState, may_have_embedded_data=true)
    at ../src/Interpreters/executeQuery.cpp:908
#3  0x000000001b7684b6 in DB::TCPHandler::runImpl (this=0x7f79b1fac000) at ../src/Server/TCPHandler.cpp:290
#4  0x000000001b775145 in DB::TCPHandler::run (this=0x7f79b1fac000) at ../src/Server/TCPHandler.cpp:1548
#5  0x000000001f075cc9 in Poco::Net::TCPServerConnection::start (this=0x7f79b1fac000) at ../contrib/poco/Net/src/TCPServerConnection.cpp:43
#6  0x000000001f0764a9 in Poco::Net::TCPServerDispatcher::run (this=0x7f79b668ce00) at ../contrib/poco/Net/src/TCPServerDispatcher.cpp:113
#7  0x000000001f1a9184 in Poco::PooledThread::run (this=0x7f7a5cd6c700) at ../contrib/poco/Foundation/src/ThreadPool.cpp:199
#8  0x000000001f1a622a in Poco::(anonymous namespace)::RunnableHolder::run (this=0x7f7a5cc59910) at ../contrib/poco/Foundation/src/Thread.cpp:55
#9  0x000000001f1a50c2 in Poco::ThreadImpl::runnableEntry (pThread=0x7f7a5cd6c738) at ../contrib/poco/Foundation/src/Thread_POSIX.cpp:345
#10 0x00007f7a5e00d6db in start_thread (arg=0x7f79ba950700) at pthread_create.c:463
#11 0x00007f7a5dd3671f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

  - 参考: <https://developer.aliyun.com/article/765184>
    - ![image2021-5-7 16:19:11.png](/assets/01KJBYDAQ6YMA4RN9VCSAXHRZ0/image2021-5-7%2016%3A19%3A11.png)

# 研究stage对执行的影响

分布式表, 远端节点收到的stage是WithMergeableState, 进行SQL解析分析后, 应仅返回数据, 不进行聚合. 需要找到stage对执行的影响机制

以"select avg(a) from t"为例: 

单机版执行计划: 

```
localhost :) explain actions=1 select avg(a) from db2.tt

EXPLAIN actions = 1
SELECT avg(a)
FROM db2.tt

Query id: 02f20f00-36fc-4cb1-8171-00fff5de9cc0

┌─explain───────────────────────────────────────────────────────────────────────┐
│ Expression ((Projection + Before ORDER BY))                                   │
│ Actions: INPUT :: 0 -> avg(a) Float64 : 0                                     │
│ Positions: 0                                                                  │
│   Aggregating                                                                 │
│   Keys:                                                                       │
│   Aggregates:                                                                 │
│       avg(a)                                                                  │
│         Function: avg(Int32) → Float64                                        │
│         Arguments: a                                                          │
│         Argument positions: 0                                                 │
│     Expression (Before GROUP BY)                                              │
│     Actions: INPUT :: 0 -> a Int32 : 0                                        │
│     Positions: 0                                                              │
│       SettingQuotaAndLimits (Set limits and quota after reading from storage) │
│         ReadFromStorage (MergeTree)                                           │
└───────────────────────────────────────────────────────────────────────────────┘
``` 

分布式版执行计划: 

```
localhost :) explain actions=1 select avg(a) from all_tt

EXPLAIN actions = 1
SELECT avg(a)
FROM all_tt

Query id: 3a2c618a-93f2-49c0-951f-33543e23e4b6

┌─explain───────────────────────────────────────────────────────────────────────────────┐
│ Expression ((Projection + Before ORDER BY))                                           │
│ Actions: INPUT :: 0 -> avg(a) Float64 : 0                                             │
│ Positions: 0                                                                          │
│   MergingAggregated                                                                   │
│   Keys:                                                                               │
│   Aggregates:                                                                         │
│       avg(a)                                                                          │
│         Function: avg(Int32) → Float64                                                │
│         Arguments: a                                                                  │
│         Argument positions: none                                                      │
│     SettingQuotaAndLimits (Set limits and quota after reading from storage)           │
│       Union                                                                           │
│         Expression (Convert block structure for query from local replica)             │
│         Actions: INPUT :: 0 -> avg(a) AggregateFunction(avg, Int32) : 0               │
│         Positions: 0                                                                  │
│           Aggregating                                                                 │
│           Keys:                                                                       │
│           Aggregates:                                                                 │
│               avg(a)                                                                  │
│                 Function: avg(Int32) → Float64                                        │
│                 Arguments: a                                                          │
│                 Argument positions: 0                                                 │
│             Expression (Before GROUP BY)                                              │
│             Actions: INPUT :: 0 -> a Int32 : 0                                        │
│             Positions: 0                                                              │
│               SettingQuotaAndLimits (Set limits and quota after reading from storage) │
│                 ReadFromStorage (MergeTree)                                           │
│         ReadFromPreparedSource (Read from remote replica)                             │
└───────────────────────────────────────────────────────────────────────────────────────┘
``` 

  - ClusterProxy/executeQuery 形成的QueryPlan: 
    - root = UnionStep
      - LocalPlan (ClusterProxy/createLocalPlan)
      - ReadFromPreparedSource
        - RemotePlan-1
        - RemotePlan-2
        - ...
  - 对于一个聚合函数, 举例定义如下: 

```
template <typename Numerator, typename Denominator, typename Derived>
class AggregateFunctionAvgBase : public
        IAggregateFunctionDataHelper<AvgFraction<Numerator, Denominator>, Derived>
```

    - 其中AvgFraction使其数据部分, Aggregating的first_stage, 聚合结果为此数据格式
      - 本例中调用add进行聚合, 在调用merge对多个线程的结果进行合并
    - Aggregating的second_stage, 通过每个AggregateFunction定义的Merge操作来完成对数据格式的合并
      - 本例中调用merge对多个节点的结果进行合并
    - 本例中, 在远程节点上, AggregateFunction的调用堆栈如下: 
      - add操作: 

```
#0  DB::AggregateFunctionAvg<int>::add (this=0x7f79b1e4bb40, place=0x7f79b8cee000 "",
    columns=0x7f79b8c36038, row_num=0) at ../src/AggregateFunctions/AggregateFunctionAvg.h:153
#1  0x00000000117b40ef in DB::IAggregateFunctionHelper<DB::AggregateFunctionAvg<int> >::addBatchSinglePlace (this=0x7f79b1e4bb40, batch_size=1, place=0x7f79b8cee000 "", columns=0x7f79b8c36038,
    arena=0x7f79b6457f98, if_argument_pos=-1) at ../src/AggregateFunctions/IAggregateFunction.h:297
#2  0x000000001a681c80 in DB::Aggregator::executeWithoutKeyImpl (
    res=@0x7f7a5d48b660: 0x7f79b8cee000 "", rows=1, aggregate_instructions=0x7f79b5237140,
    arena=0x7f79b6457f98) at ../src/Interpreters/Aggregator.cpp:572
#3  0x000000001a682eee in DB::Aggregator::executeOnBlock (this=0x7f79b2061d40, columns=...,
    num_rows=1, result=..., key_columns=..., aggregate_columns=...,
    no_more_keys=@0x7f7a5d4d6068: false) at ../src/Interpreters/Aggregator.cpp:716
#4  0x000000001bb54375 in DB::AggregatingTransform::consume (this=0x7f7a5d4d5f98, chunk=...)
    at ../src/Processors/Transforms/AggregatingTransform.cpp:525
#5  0x000000001bb526cc in DB::AggregatingTransform::work (this=0x7f7a5d4d5f98)
    at ../src/Processors/Transforms/AggregatingTransform.cpp:495
#6  0x000000001b814da9 in DB::executeJob (processor=0x7f7a5d4d5f98)
    at ../src/Processors/Executors/PipelineExecutor.cpp:79
#7  0x000000001b814d1f in DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0::operator()() const (this=0x7f79b1eb11b0) at ../src/Processors/Executors/PipelineExecutor.cpp:96
``` 
      - merge操作

```
#0  DB::AggregateFunctionAvgBase<long, unsigned long, DB::AggregateFunctionAvg<int> >::merge (
    this=0x7f79b1e4bb40, place=0x7f79b8cee000 "\001", rhs=0x7f79abc86000 "\005")
    at ../src/AggregateFunctions/AggregateFunctionAvg.h:103
#1  0x000000001a68c0be in DB::Aggregator::mergeWithoutKeyDataImpl (this=0x7f79b2061d40,
    non_empty_data=...) at ../src/Interpreters/Aggregator.cpp:1635
#2  0x000000001bb5bee5 in DB::ConvertingAggregatedToChunksTransform::initialize (
    this=0x7f79b8dde018) at ../src/Processors/Transforms/AggregatingTransform.cpp:340
#3  0x000000001bb58a3f in DB::ConvertingAggregatedToChunksTransform::work (this=0x7f79b8dde018)
    at ../src/Processors/Transforms/AggregatingTransform.cpp:175
#4  0x000000001b814da9 in DB::executeJob (processor=0x7f79b8dde018)
    at ../src/Processors/Executors/PipelineExecutor.cpp:79
#5  0x000000001b814d1f in DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0::operator()() const (this=0x7f79b8d11bb0) at ../src/Processors/Executors/PipelineExecutor.cpp:96
...
```

    - 在本地节点进行结果聚合的堆栈: 

```
#0  DB::AggregateFunctionAvgBase<long, unsigned long, DB::AggregateFunctionAvg<int> >::merge (
    this=0x7f79b524f280, place=0x7f7a5b9bc000 "", rhs=0x7f7a5b9cf000 "\006")
    at ../src/AggregateFunctions/AggregateFunctionAvg.h:103
#1  0x000000001a68cf0e in DB::Aggregator::mergeWithoutKeyStreamsImpl (this=0x7f79b53e8940,
    block=..., result=...) at ../src/Interpreters/Aggregator.cpp:1879
#2  0x000000001a6911d4 in DB::Aggregator::mergeBlocks (this=0x7f79b53e8940, blocks=..., final=true)
    at ../src/Interpreters/Aggregator.cpp:2081
#3  0x000000001bacf31e in DB::MergingAggregatedBucketTransform::transform (this=0x7f7a5b9e12d8,
    chunk=...) at ../src/Processors/Transforms/MergingAggregatedMemoryEfficientTransform.cpp:344
#4  0x000000001bab9312 in DB::ISimpleTransform::transform (this=0x7f7a5b9e12d8, input_chunk=...,
    output_chunk=...) at ../src/Processors/ISimpleTransform.h:42
#5  0x000000001babca4b in DB::ISimpleTransform::work (this=0x7f7a5b9e12d8)
    at ../src/Processors/ISimpleTransform.cpp:89
#6  0x000000001b814da9 in DB::executeJob (processor=0x7f7a5b9e12d8)
    at ../src/Processors/Executors/PipelineExecutor.cpp:79
#7  0x000000001b814d1f in DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0::operator()() const (this=0x7f7a5b9e2230) at ../src/Processors/Executors/PipelineExecutor.cpp:96
```

      - 由MergeAggregated动作完成
  - InterpreterSelectQuery::executeImpl的逻辑: 
    - 对于分布式Query
      - 远端节点的参数为: first_stage=true, second_stage=false, aggregate_final=false
      - 远端节点根据参数执行聚合, 但不做最终汇总
      - 近端节点的参数为:first_stage = false, second_stage = true, aggregate_final = true, second_stage时才会生成MergeAggregated动作
        - 在second_stage中, 生成分布式执行计划时, 会再次调用InterpreterSelectQuery::executeImpl生成获取本地数据的逻辑, 此时参数为: first_stage = true, second_stage = false, aggregate_final=false, 与远端节点相同
    - 对于单机Query:
      - 参数为: first_stage=true, second_stage=true, aggregate_final=true
      - 在first_stage中, 进行聚合时由于aggregate_final, 直接进行汇总
      - 在second_stage中, 由于first_stage=true, 跳过MergeAggregated (first_stage和second_stage同时为true, 代表是单机Query??)

# 其他

  - clickhouse节点均需配置zookeeper, 否则进行CREATE TABLE 创建分布式表时, 命令会阻塞 (猜测命令在等待另一节点更新 zookeeper的标志位)
  - 关于only_types的说明: 

If only_types = true set, does not execute subqueries in the relevant parts of the query. The actions got this way  
shouldn't be executed, they are only needed to get a list of columns with their types.

# 线索

  - 分析group by在单机和分布式上的逻辑区别

# TODO

insert_distributed_sync

engine的参数: 

policy name

fsync_after_insert

fsync_directories

bytes_to_throw_insert

bytes_to_delay_insert

max_delay_to_insert

replica.priority

compression

关于 Durability settings 的 Note ??

distributed_replica_max_ignored_errors

max_parallel_replicas

distributed_group_by_no_merge

optimize_aggregation_in_order

size_limits_for_set

如何更新config.xml中的shard节点配置使其生效

TOTALS / ROLLUP / CUBE
