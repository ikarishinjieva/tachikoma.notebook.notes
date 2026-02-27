---
title: 20210415 - Clickhouse MaterializedMySQL 学习
confluence_page_id: 753972
created_at: 2021-04-15T05:47:12+00:00
updated_at: 2021-04-21T07:36:36+00:00
---

# 文档

  - <https://clickhouse.tech/docs/en/engines/database-engines/materialize-mysql/>
    - 虚拟列
      - _version: Transaction counter, 事务计数器?
      - _sign: 删除标志: -1 表示行删除
    - Data Types Support: MySQL数据类型与Clickhouse数据类型的映射表

      - 如果遇到不支持的column类型, 复制会因异常中断
    - DDL: 
      - MySQL DDL会被转换成相应的DDL: ALTER, CREATE, DROP, RENAME
      - 不能识别的DDL, 会直接被忽略
    - DML
      - MySQL INSERT, 会转换成 INSERT with _sign=1
      - MySQL DELETE, 会转换成 INSERT with _sign=-1
      - MySQL UPDATE, 会转换成 INSERT with _sign=-1 and INSERT with _sign=1 (先删除再插入)
    - SELECT
      - 如果SELECT没有指定 _version, 则转换成 FINAL 形式, 选取MAX(_version) 的行
      - 如果SELECT没有指定 _sign, 则转换成 _sign=1, 排除已删除行
    - PRIMARY KEY/INDEX, 转换成 ORDER BY
    - 受到 optimize_on_insert 影响
      - optimize_on_insert: insert前, 进行数据合并

# 代码入口

  - InterpreterCreateQuery::createDatabase
    - database->loadStoredObjects
      - materialize_thread.startSynchronization()
        - MaterializeMySQLSyncThread::synchronization
          - 建立复制: MaterializeMySQLSyncThread::prepareSynchronized
            - MaterializeMetadata::MaterializeMetadata
              - 开启事务
              - 获取CREATE TABLE语句
            - 清理 outdated tables??
            - dumpDataForTables
              - "Materialize MySQL step 1: execute MySQL DDL for dump data"
                - 执行create table
              - ""Materialize MySQL step 1: execute dump data"
                - 源端执行: SELECT * FROM ...
                - 目标端执行: INSERT INTO ... VALUES (...)
              - 每张表顺序进行
            - 提交事务
            - client.connect()
            - client.startBinlogDumpGTID()
              - SET @master_binlog_checksum = 'CRC32'
              - SET @master_heartbeat_period = ???
              - 随机生成server id 
              - 指定replicate db
              - 使用GTID对位
          - 开始复制
            - MaterializeMySQLSyncThread::onEvent 将 binlog event 送入buffer
              - buffer的结构: map<表名, 每张表的缓存>, 每张表的缓存 = std::pair> = <缓存Block, 排序列的位置序列>
              - 分event处理:
                - MYSQL_WRITE_ROWS_EVENT/MYSQL_UPDATE_ROWS_EVENT/MYSQL_DELETE_ROWS_EVENT
                  - 组织成 列存储, 并增加sign和version列
                - MYSQL_QUERY_EVENT
                  - DDL前先提交buffer
                  - MaterializeMySQLSyncThread::executeDDLAtomic
                    - "Materialize MySQL step 2: execute MySQL DDL for sync data"
                    - 若发生语法错误, 则抛出异常. 否则跳过异常

                - ROTATE_EVENT
                  - 直接从master获取binlog_checksum, 以更新
                    - 当复制存在延迟时, 此操作会获取最新值, 而非binlog的当时值, 存在状态不一致的可能
                - 其他event类型, 排除HEARTBEAT_EVENT, 其他类型都打印DEBUG日志
              - buffers.add: 用于更新buffer的统计信息
            - 当 间隔时间超过 max_flush_time, 或大小超过阈值 (max_rows_in_buffer, max_rows_in_buffers, max_bytes_in_buffer, max_bytes_in_buffers), 对buffer进行flush
              - MaterializeMySQLSyncThread::flushBuffersData
                - MaterializeMySQLSyncThread::Buffers::commit
                  - 将buffer中构造好的block, 复制到表中
                  - 复制源 (IBlockInputStream) 为buffer中的block
                  - 复制目标 (IBlockOutputStream) 为表, 链条为 (从外往内): 
                    - CountingBlockOutputStream: 统计progress用

                    - SquashingBlockOutputStream: 整形用, 将连续的block整形成合适的大小

                    - AddingDefaultBlockOutputStream: 填补源和目标之间确实的列

                    - PushingToViewsBlockOutputStream: 为后续的materialize view和后续表生成数据

                    - MergeTreeBlockOutputStream: 将block填入表

# 如何查看复制进度

```
root@R740-24:/var/lib/clickhouse/metadata/atest# cat .metadata
Version:	2
Binlog File:	mysql-bin.000002
Executed GTID:	00019327-1111-1111-1111-111111111111:1-22
Binlog Position:	1039
``` 

# 如何找到database的uuid

  - 查找文件

```
root@ubuntu:/var/lib/clickhouse/metadata# cat atest.sql
ATTACH DATABASE _ UUID 'e1ddb10d-9a4b-4908-b824-5748c40ebc43'
ENGINE = MaterializeMySQL('127.0.0.1:19327', 'test', 'rsandbox', 'rsandbox')
```

  - 查找元数据表: select * from databases;

# 如何进行故障恢复

  -     - 连接错误: 
      - 预同步和同步阶段: 如果是connection错误, 会等待max_wait_time_when_mysql_unavailable时间, 以防冲击MySQL
      - 在某版本后, 才有自动重试的功能. (目前使用的版本为21.4.4.30)
    - 非连接错误: 
      - 重启复制: detach + attach
      - 如何修复 由于数据不一致 引起的复制错误
        - detach
        - ATTACH DATABASE test UUID 'e1ddb10d-9a4b-4908-b824-5748c40ebc43' ENGINE = Atomic
        - 修改数据
        - detach
        - ATTACH DATABASE test UUID 'e1ddb10d-9a4b-4908-b824-5748c40ebc43'  
ENGINE = MaterializeMySQL('127.0.0.1:19327', 'testb', 'rsandbox', 'rsandbox')

# 如何查看复制状态

  - 如果复制状态异常, 对表进行select时就会报错
    - 举例: 

```
ubuntu :) select * from a;

SELECT *
FROM a

Query id: 7efc8b71-26db-40cd-9063-3cc7e1a4ee3d

0 rows in set. Elapsed: 0.002 sec.

Received exception from server (version 21.4.4):
Code: 1000. DB::Exception: Received from localhost:9000. DB::Exception: mysqlxx::ConnectionFailed: Can't connect to MySQL server on '127.0.0.1' (115) ((nullptr):0).

``` 
    - 处理SQL的线程为什么会扔出同步线程的错误, 处理堆栈如下: 

```
#0  DB::StorageMaterializeMySQL::getVirtuals (this=0x7f71eba32f58)
    at ../src/Storages/StorageMaterializeMySQL.cpp:109
#1  0x000000001adb7d93 in DB::getColumnsFromTableExpression (table_expression=...,
    context=..., materialized=..., aliases=..., virtuals=...)
    at ../src/Interpreters/getTableExpressions.cpp:110
#2  0x000000001adb802b in DB::getDatabaseAndTablesWithColumns (table_expressions=...,
    context=...) at ../src/Interpreters/getTableExpressions.cpp:139
#3  0x000000001abcc49d in DB::JoinedTables::resolveTables (this=0x7f71f473b750)
    at ../src/Interpreters/JoinedTables.cpp:191
#4  0x000000001a60d80a in DB::InterpreterSelectQuery::InterpreterSelectQuery (
    this=0x7f71ee981000, query_ptr_=..., context_=..., input_=..., input_pipe_=...,
    storage_=..., options_=..., required_result_column_names=...,
    metadata_snapshot_=...) at ../src/Interpreters/InterpreterSelectQuery.cpp:329
#5  0x000000001a60cc74 in DB::InterpreterSelectQuery::InterpreterSelectQuery (
    this=0x7f71ee981000, query_ptr_=..., context_=..., options_=...,
    required_result_column_names_=...)
    at ../src/Interpreters/InterpreterSelectQuery.cpp:160
#6  0x000000001ab87f39 in std::__1::make_unique<DB::InterpreterSelectQuery, std::__1::shared_ptr<DB::IAST> const&, DB::Context&, DB::SelectQueryOptions&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&> (__args=..., __args=..., __args=..., __args=...)
    at ../contrib/libcxx/include/memory:2068
#7  0x000000001ab86200 in DB::InterpreterSelectWithUnionQuery::buildCurrentChildInterpreter (this=0x7f71eba23380, ast_ptr_=..., current_required_result_column_names=...)
    at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:211
#8  0x000000001ab85b5d in DB::InterpreterSelectWithUnionQuery::InterpreterSelectWithUnionQuery (this=0x7f71eba23380, query_ptr_=..., context_=..., options_=...,
    required_result_column_names=...)
    at ../src/Interpreters/InterpreterSelectWithUnionQuery.cpp:133
#9  0x000000001a5a9c94 in std::__1::make_unique<DB::InterpreterSelectWithUnionQuery, std::__1::shared_ptr<DB::IAST>&, DB::Context&, DB::SelectQueryOptions const&> (__args=...,
    __args=..., __args=...) at ../contrib/libcxx/include/memory:2068
#10 0x000000001a5a8338 in DB::InterpreterFactory::get (query=..., context=...,
    options=...) at ../src/Interpreters/InterpreterFactory.cpp:110
#11 0x000000001ada23ed in DB::executeQueryImpl (
    begin=0x7f71ebbf7b30 "select * from a;", end=0x7f71ebbf7b40 "", context=...,
    internal=false, stage=DB::QueryProcessingStage::Complete, has_query_tail=false,
    istr=0x0) at ../src/Interpreters/executeQuery.cpp:520
#12 0x000000001ada0c0a in DB::executeQuery (query=..., context=..., internal=false,
    stage=DB::QueryProcessingStage::Complete, may_have_embedded_data=true)
    at ../src/Interpreters/executeQuery.cpp:908
#13 0x000000001b7684b6 in DB::TCPHandler::runImpl (this=0x7f71ebbf4800)
    at ../src/Server/TCPHandler.cpp:290
---Type <return> to continue, or q <return> to quit---
#14 0x000000001b775145 in DB::TCPHandler::run (this=0x7f71ebbf4800)
    at ../src/Server/TCPHandler.cpp:1548
#15 0x000000001f075cc9 in Poco::Net::TCPServerConnection::start (this=0x7f71ebbf4800)
    at ../contrib/poco/Net/src/TCPServerConnection.cpp:43
#16 0x000000001f0764a9 in Poco::Net::TCPServerDispatcher::run (this=0x7f71f2247300)
    at ../contrib/poco/Net/src/TCPServerDispatcher.cpp:113
#17 0x000000001f1a9184 in Poco::PooledThread::run (this=0x7f729636c480)
    at ../contrib/poco/Foundation/src/ThreadPool.cpp:199
#18 0x000000001f1a622a in Poco::(anonymous namespace)::RunnableHolder::run (
    this=0x7f7296259880) at ../contrib/poco/Foundation/src/Thread.cpp:55
#19 0x000000001f1a50c2 in Poco::ThreadImpl::runnableEntry (pThread=0x7f729636c4b8)
    at ../contrib/poco/Foundation/src/Thread_POSIX.cpp:345
#20 0x00007f72976b36db in start_thread (arg=0x7f71f4751700) at pthread_create.c:463
#21 0x00007f72973dc71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

在rethrowExceptionIfNeed的处理中, 发现行为与allows_query_when_mysql_lost配置有关

  - 如果开启allows_query_when_mysql_lost, 如何查看复制状态
    - 通过日志 ??

# 如何在复制源断开后, 仍保持数据有效

开启allows_query_when_mysql_lost

```
ATTACH DATABASE test UUID 'e1ddb10d-9a4b-4908-b824-5748c40ebc43'
ENGINE = MaterializeMySQL('127.0.0.1:19327', 'testb', 'rsandbox', 'rsandbox')
SETTINGS allows_query_when_mysql_lost=1
``` 

# 如何改变复制来源

```
detach
ATTACH DATABASE test UUID 'e1ddb10d-9a4b-4908-b824-5748c40ebc43'
ENGINE = MaterializeMySQL('127.0.0.1:19327', 'testb', 'rsandbox', 'rsandbox')
``` 

# 复制的位点与事务的关系

  - Clickhouse仅读取row events 和 Query event
  - 碰到Query event时, 会更新GTID (有可能漏掉最后一个事务的GTID)
  - 会忽略 XID_event/Table_map_event等, 日志中有如下信息: 

```
2021.04.21 05:32:55.284132 [ 24575 ] {} <Debug> MaterializeMySQLSyncThread: Skip MySQL event:

=== TableMapEvent ===
Timestamp: 1618983172
Event Type: TableMapEvent
Server ID: 19327
Event Size: 45
Log Pos: 632
Flags: 0
Table ID: 109
Flags: 1
Schema Len: 5
Schema: testb
Table Len: 1
Table: a
Column Count: 1
Column Type [0]: 3, Meta: 0
Null Bitmap: 00000000
 
2021.04.21 05:32:55.284257 [ 24575 ] {} <Debug> MaterializeMySQLSyncThread: Skip MySQL event:

=== XIDEvent ===
Timestamp: 1618983175
Event Type: XIDEvent
Server ID: 19327
Event Size: 31
Log Pos: 703
Flags: 0
XID: 61
``` 

# 如何避开全量阶段, 从增量开始复制

  - 建立普通database, 创建数据表是需带有_version和_sign字段, 并带有相关索引: 

```
CREATE TABLE ddd.a
(
    `a` Int32,
    `_sign` Int8 MATERIALIZED 1,
    `_version` UInt64 MATERIALIZED 1,
    INDEX _version _version TYPE minmax GRANULARITY 1
)
ENGINE = ReplacingMergeTree(_version)
PARTITION BY intDiv(a, 4294967)
ORDER BY tuple(a)
SETTINGS index_granularity = 8192
``` 
  - 灌入数据
  - detach database
  - 创建/var/lib/clickhouse/metadata/bbb/.metadata, 内容如下 (其中键与值之间的分隔符需改为制表符, 而不是空格):   
  

```
Version:        2
Binlog File:    mysql-bin.000006
Executed GTID:  00019327-1111-1111-1111-111111111111:1-36
Binlog Position:        1010
Data Version:   1
``` 

    - Binlog File 用于判断是否进行全量复制 (如果show master logs中包含Binlog File, 则跳过全量复制)
    - GTID用于进行复制对位, 其他可以随便填写
  - attach database 

ENGINE = MaterializeMySQL('127.0.0.1:19327', 'empty', 'rsandbox', 'rsandbox')  
SETTINGS allows_query_when_mysql_lost=1

# 问题

  -     - 集群模式下, 复制关系是怎样的

    - 复制效率
