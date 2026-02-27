---
title: 20221020 - 光大clickhouse Too many parts报错
confluence_page_id: 1934312
created_at: 2022-10-20T03:24:51+00:00
updated_at: 2022-10-31T16:36:04+00:00
---

# 原始工单

<https://support.actionsky.com/service_desk/browse/BEIJ-2780>

传入两台CK的日志: 26和27

# 27日志分析 - AST is too big 报错

27日志中, 明确在 复制表中 报错: AST is too big, 会阻塞复制过程. 27中的数据会产生延迟.

```
2022.09.01 08:05:04.748311 [ 15 ] {} <Error> default.CK_BLACK_IP_LOCAL: DB::StorageReplicatedMergeTree::queueTask()::<lambda(DB::StorageReplicatedMergeTree::LogEntryPtr&)>: Code: 168, e.displayText() = DB::Exception: AST is too big. Maximum: 500000: (after expansion of aliases), Stack trace (when copying this message, always include the lines below):
0. 0xb2087bc Poco::Exception::Exception(std::_1::basic_string<char, std::1::char_traits<char>, std::_1::allocator<char> > const&, int) in /data/stix/clickhouse/bin/clickhouse
1. 0x4d8e3c9 DB::Exception::Exception(std::_1::basic_string<char, std::1::char_traits<char>, std::_1::allocator<char> > const&, int) in /data/stix/clickhouse/bin/clickhouse
2. 0x9173c04 DB::IAST::checkSize(unsigned long) const in /data/stix/clickhouse/bin/clickhouse
3. 0x9173b80 DB::IAST::checkSize(unsigned long) const in /data/stix/clickhouse/bin/clickhouse
4. 0x9173b80 DB::IAST::checkSize(unsigned long) const in /data/stix/clickhouse/bin/clickhouse
5. 0x8f640c5 DB::QueryNormalizer::visit(std::__1::shared_ptr<DB::IAST>&, DB::QueryNormalizer::Data&) in /data/stix/clickhouse/bin/clickhouse
6. 0x8633694 DB::SyntaxAnalyzer::analyze(std::_1::shared_ptr<DB::IAST>&, DB::NamesAndTypesList const&, std::1::vector<std::1::basic_string<char, std::1::char_traits<char>, std::1::allocator<char> >, std::1::allocator<std::1::basic_string<char, std::1::char_traits<char>, std::1::allocator<char> > > > const&, std::_1::shared_ptr<DB::IStorage>, DB::NamesAndTypesList const&) const in /data/stix/clickhouse/bin/clickhouse
7. 0x84d6d5d ? in /data/stix/clickhouse/bin/clickhouse
8. 0x84d84c4 DB::InterpreterSelectQuery::InterpreterSelectQuery(std::_1::shared_ptr<DB::IAST> const&, DB::Context const&, std::1::shared_ptr<DB::IBlockInputStream> const&, std::1::shared_ptr<DB::IStorage> const&, DB::SelectQueryOptions const&, std::1::vector<std::1::basic_string<char, std::1::char_traits<char>, std::1::allocator<char> >, std::1::allocator<std::1::basic_string<char, std::1::char_traits<char>, std::_1::allocator<char> > > > const&) in /data/stix/clickhouse/bin/clickhouse
9. 0x84d96c9 DB::InterpreterSelectQuery::InterpreterSelectQuery(std::_1::shared_ptr<DB::IAST> const&, DB::Context const&, std::_1::shared_ptr<DB::IStorage> const&, DB::SelectQueryOptions const&) in /data/stix/clickhouse/bin/clickhouse
10. 0x8f4b29d DB::isStorageTouchedByMutations(std::_1::shared_ptr<DB::IStorage>, std::1::vector<DB::MutationCommand, std::_1::allocator<DB::MutationCommand> > const&, DB::Context) in /data/stix/clickhouse/bin/clickhouse
11. 0x88c3549 DB::MergeTreeDataMergerMutator::mutatePartToTemporaryPart(DB::FutureMergedMutatedPart const&, std::_1::vector<DB::MutationCommand, std::1::allocator<DB::MutationCommand> > const&, DB::MergeListEntry&, DB::Context const&, std::1::unique_ptr<DB::IReservation, std::_1::default_delete<DB::IReservation> > const&, DB::TableStructureReadLockHolder&) in /data/stix/clickhouse/bin/clickhouse
12. 0x87e8862 DB::StorageReplicatedMergeTree::tryExecutePartMutation(DB::ReplicatedMergeTreeLogEntry const&) in /data/stix/clickhouse/bin/clickhouse
13. 0x8811a39 DB::StorageReplicatedMergeTree::executeLogEntry(DB::ReplicatedMergeTreeLogEntry&) in /data/stix/clickhouse/bin/clickhouse
14. 0x88129dc ? in /data/stix/clickhouse/bin/clickhouse
15. 0x8990bbe DB::ReplicatedMergeTreeQueue::processEntry(std::_1::function<std::1::shared_ptr<zkutil::ZooKeeper> ()>, std::1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&, std::1::function<bool (std::_1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&)>) in /data/stix/clickhouse/bin/clickhouse
16. 0x87d255f DB::StorageReplicatedMergeTree::queueTask() in /data/stix/clickhouse/bin/clickhouse
17. 0x885778d DB::BackgroundProcessingPool::threadFunction() in /data/stix/clickhouse/bin/clickhouse
18. 0x885819c ? in /data/stix/clickhouse/bin/clickhouse
19. 0x4dc7eca ThreadPoolImpl<std::_1::thread>::worker(std::1::list_iterator<std::_1::thread, void*>) in /data/stix/clickhouse/bin/clickhouse
20. 0x4dc69dc ? in /data/stix/clickhouse/bin/clickhouse
21. 0x7dd5 start_thread in /usr/lib64/libpthread-2.17.so
22. 0xfeb3d __clone in /usr/lib64/libc-2.17.so
(version 20.1.8.41)
``` 

### 模拟报错

CK实例A和B, 建立复制

在节点A, 配置 max_ast_elements 和 max_expanded_ast_elements, 方便复现

```
    <system_profile>sys</system_profile>
    <profiles>
...
        <sys>
                <max_ast_elements>50</max_ast_elements>
                <max_expanded_ast_elements>50</max_expanded_ast_elements>
        </sys>
...
    </profiles>
``` 

复制表结构

```
CREATE TABLE default.table_test4 (`EventDate` DateTime, `CounterID` UInt32, `UserID` UInt32, `AAA` UInt32) 
	ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/table_test4', '{replica}') 
	PARTITION BY toYYYYMM(EventDate) 
	ORDER BY (CounterID, EventDate, intHash32(UserID)) 
	SAMPLE BY intHash32(UserID) 
	SETTINGS index_granularity = 8192;
 
insert into table_test4 values('2019-01-01 00:00:04',0,0,0);
insert into table_test4 values('2019-01-02 00:00:04',0,0,0);
insert into table_test4 values('2019-01-03 00:00:04',0,0,0);
insert into table_test4 values('2019-01-04 00:00:04',0,0,0);
insert into table_test4 values('2019-01-05 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-02-01 00:00:04',0,0,0);
insert into table_test4 values('2019-03-01 00:00:04',0,0,0);
insert into table_test4 values('2019-04-01 00:00:04',0,0,0);
``` 

在CK-B上执行 多个alter table, 使得mutation堆积到A: 

```
alter table table_test4 update AAA =1 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04'; alter table table_test4 update AAA =2 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04';alter table table_test4 update AAA =3 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04';alter table table_test4 update AAA =4 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04';alter table table_test4 update AAA =5 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04'; alter table table_test4 update AAA =6 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04';alter table table_test4 update AAA =7 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04';alter table table_test4 update AAA =8 where EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04' AND EventDate = '2019-01-01 00:00:04';
``` 

可以观察到 A 的复制的报错: 

```
2022.10.20 07:34:14.110199 [ 22 ] {} <Error> default.table_test4: auto DB::StorageReplicatedMergeTree::queueTask()::(anonymous class)::operator()(DB::StorageReplicatedMergeTree::LogEntryPtr &) const: Code: 168, e.displayText() = DB::Exception: AST is too big. Maximum: 50: (after expansion of aliases), Stack trace (when copying this message, always include the lines below):

0. 0xb8f9b58 std::exception::capture() /opt/ClickHouse/build/../contrib/libcxx/include/exception:129 in /opt/ClickHouse/build/dbms/programs/clickhouse
1. 0x1d0cdd99 Poco::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int) /opt/ClickHouse/build/../contrib/poco/Foundation/src/Exception.cpp:28 in /opt/ClickHouse/build/d
bms/programs/clickhouse
2. 0xb8fde3d DB::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int) /opt/ClickHouse/build/../dbms/src/Common/Exception.cpp:35 in /opt/ClickHouse/build/dbms/programs/cl
ickhouse
3. 0x1864dc8d DB::IAST::checkSize(unsigned long) const /opt/ClickHouse/build/../dbms/src/Parsers/IAST.cpp:45 in /opt/ClickHouse/build/dbms/programs/clickhouse
4. 0x1864db2c DB::IAST::checkSize(unsigned long) const /opt/ClickHouse/build/../dbms/src/Parsers/IAST.cpp:42 in /opt/ClickHouse/build/dbms/programs/clickhouse
5. 0x1864db2c DB::IAST::checkSize(unsigned long) const /opt/ClickHouse/build/../dbms/src/Parsers/IAST.cpp:42 in /opt/ClickHouse/build/dbms/programs/clickhouse
6. 0x180484ea DB::QueryNormalizer::visit(std::__1::shared_ptr<DB::IAST>&, DB::QueryNormalizer::Data&) /opt/ClickHouse/build/../dbms/src/Interpreters/QueryNormalizer.cpp:239 in /opt/ClickHouse/build/dbms/programs/clickhouse
7. 0x1679fb40 DB::QueryNormalizer::visit(std::__1::shared_ptr<DB::IAST>&) /opt/ClickHouse/build/../dbms/src/Interpreters/QueryNormalizer.h:61 in /opt/ClickHouse/build/dbms/programs/clickhouse
8. 0x16786b53 DB::SyntaxAnalyzer::analyze(std::__1::shared_ptr<DB::IAST>&, DB::NamesAndTypesList const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__
1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::shared_ptr<DB::IStorage>, DB::NamesAndTypesList const&) const /opt/ClickHouse/build/../dbms/src/Interpreters/SyntaxAnalyzer.cpp:902 in
/opt/ClickHouse/build/dbms/programs/clickhouse
9. 0x16020887 DB::InterpreterSelectQuery::InterpreterSelectQuery(std::__1::shared_ptr<DB::IAST> const&, DB::Context const&, std::__1::shared_ptr<DB::IBlockInputStream> const&, std::__1::shared_ptr<DB::IStorage> const&, DB::SelectQueryOptions const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&)::$_1::operator()(bool) const /opt/ClickHouse/build/../dbms/src/Interpreters/InterpreterSelectQuery.cpp:320 in /opt/ClickHouse/build/dbms/programs/clickhouse
10. 0x1601b9b9 DB::InterpreterSelectQuery::InterpreterSelectQuery(std::__1::shared_ptr<DB::IAST> const&, DB::Context const&, std::__1::shared_ptr<DB::IBlockInputStream> const&, std::__1::shared_ptr<DB::IStorage> const&, DB::SelectQueryOptions const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&) /opt/ClickHouse/build/../dbms/src/Interpreters/InterpreterSelectQuery.cpp:394 in /opt/ClickHouse/build/dbms/programs/clickhouse
11. 0x1601f185 DB::InterpreterSelectQuery::InterpreterSelectQuery(std::__1::shared_ptr<DB::IAST> const&, DB::Context const&, std::__1::shared_ptr<DB::IStorage> const&, DB::SelectQueryOptions const&) /opt/ClickHouse/build/../dbms/src/Interpreters/InterpreterSelectQuery.cpp:189 in /opt/ClickHouse/build/dbms/programs/clickhouse
12. 0x17fcfa1d DB::isStorageTouchedByMutations(std::__1::shared_ptr<DB::IStorage>, std::__1::vector<DB::MutationCommand, std::__1::allocator<DB::MutationCommand> > const&, DB::Context) /opt/ClickHouse/build/../dbms/src/Interpreters/MutationsInterpreter.cpp:151 in /opt/ClickHouse/build/dbms/programs/clickhouse
13. 0x1701588d DB::MergeTreeDataMergerMutator::mutatePartToTemporaryPart(DB::FutureMergedMutatedPart const&, std::__1::vector<DB::MutationCommand, std::__1::allocator<DB::MutationCommand> > const&, DB::MergeListEntry&, DB::Context const&, std::__1::unique_ptr<DB::IReservation, std::__1::default_delete<DB::IReservation> > const&, DB::TableStructureReadLockHolder&) /opt/ClickHouse/build/../dbms/src/Storages/MergeTree/MergeTreeDataMergerMutator.cpp:959 in /opt/ClickHouse/build/dbms/programs/clickhouse
14. 0x16c912df DB::StorageReplicatedMergeTree::tryExecutePartMutation(DB::ReplicatedMergeTreeLogEntry const&) /opt/ClickHouse/build/../dbms/src/Storages/StorageReplicatedMergeTree.cpp:1225 in /opt/ClickHouse/build/dbms/programs/clickhouse
15. 0x16c6e284 DB::StorageReplicatedMergeTree::executeLogEntry(DB::ReplicatedMergeTreeLogEntry&) /opt/ClickHouse/build/../dbms/src/Storages/StorageReplicatedMergeTree.cpp:978 in /opt/ClickHouse/build/dbms/programs/clickhouse
16. 0x16d8826b DB::StorageReplicatedMergeTree::queueTask()::$_15::operator()(std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&) const /opt/ClickHouse/build/../dbms/src/Storages/StorageReplicatedMergeTree.cpp:2158 in /opt/ClickHouse/build/dbms/programs/clickhouse
17. 0x16d881fd bool std::__1::__invoke_void_return_wrapper<bool>::__call<DB::StorageReplicatedMergeTree::queueTask()::$_15&, std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&>(DB::StorageReplicatedMergeTree::queueTask()::$_15&, std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&) /opt/ClickHouse/build/../contrib/libcxx/include/__functional_base:317 in /opt/ClickHouse/build/dbms/programs/clickhouse
18. 0x16d880f1 std::__1::__function::__func<DB::StorageReplicatedMergeTree::queueTask()::$_15, std::__1::allocator<DB::StorageReplicatedMergeTree::queueTask()::$_15>, bool (std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&)>::operator()(std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&) /opt/ClickHouse/build/../contrib/libcxx/include/functional:1714 in /opt/ClickHouse/build/dbms/programs/clickhouse
19. 0x172b20c9 std::__1::function<bool (std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&)>::operator()(std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&) const /opt/ClickHouse/build/../contrib/libcxx/include/functional:2473 in /opt/ClickHouse/build/dbms/programs/clickhouse
20. 0x1728802a DB::ReplicatedMergeTreeQueue::processEntry(std::__1::function<std::__1::shared_ptr<zkutil::ZooKeeper> ()>, std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&, std::__1::function<bool (std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&)>) /opt/ClickHouse/build/../dbms/src/Storages/MergeTree/ReplicatedMergeTreeQueue.cpp:1148 in /opt/ClickHouse/build/dbms/programs/clickhouse
21. 0x16ce0859 DB::StorageReplicatedMergeTree::queueTask() /opt/ClickHouse/build/../dbms/src/Storages/StorageReplicatedMergeTree.cpp:2154 in /opt/ClickHouse/build/dbms/programs/clickhouse
22. 0x16d91ec8 DB::StorageReplicatedMergeTree::startup()::$_21::operator()() const /opt/ClickHouse/build/../dbms/src/Storages/StorageReplicatedMergeTree.cpp:2926 in /opt/ClickHouse/build/dbms/programs/clickhouse
 
...
``` 

相关part的数据会维持在中间状态, 不进行更新: 

```
mysql> select * from table_test4;
+---------------------+-----------+--------+------+
| EventDate           | CounterID | UserID | AAA  |
+---------------------+-----------+--------+------+
| 2019-02-01 00:00:04 |         0 |      0 |    0 |
| 2019-03-01 00:00:04 |         0 |      0 |    0 |
| 2019-01-01 00:00:04 |         0 |      0 |    6 |  //完全更新后, 此处数据应为8
| 2019-01-02 00:00:04 |         0 |      0 |    0 |
| 2019-01-03 00:00:04 |         0 |      0 |    0 |
| 2019-01-04 00:00:04 |         0 |      0 |    0 |
| 2019-01-05 00:00:04 |         0 |      0 |    0 |
| 2019-01-06 00:00:04 |         0 |      0 |    0 |
| 2019-04-01 00:00:04 |         0 |      0 |    0 |
+---------------------+-----------+--------+------+
9 rows in set (0.01 sec)
Read 9 rows, 144.00 B in 0.012 sec., 733 rows/sec., 11.46 KiB/sec.
``` 

该复制线程会在10min内重启任务, 其他复制线程会正常工作, 直到碰到 同一表的mutations, 再次陷入报错

该状态会持续到某个操作触发了part 被重新复制, 使得出错的mutation被跳过

如在CK-B上执行

```
alter table table_test4 update AAA=300 where 1=1;
``` 

在CK-A上会出现如下日志: 

```
...

2022.10.20 07:49:09.083986 [ 17 ] {} <Debug> default.table_test4: Cloning part /opt/clickhouse-home/data/data/default/table_test4/201903_266_266_0_274/ to /opt/clickhouse-home/data/data/default/table_test4/tmp_clone_201903_266_266_0_282
2022.10.20 07:49:09.086383 [ 17 ] {} <Trace> default.table_test4: Renaming temporary part tmp_clone_201903_266_266_0_282 to 201903_266_266_0_282.
 
...
2022.10.20 07:49:10.178800 [ 32 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Marking mutation 0000000296 done because it is <= mutation_pointer (0000000297)
2022.10.20 07:49:10.178875 [ 32 ] {} <Information> default.table_test4 (ReplicatedMergeTreeQueue): Seems like we jumped over mutation 0000000296 when downloaded part with bigger mutation number. It's OK, tasks for rest parts will be ski
pped, but probably a lot of mutations were executed concurrently on different replicas.
2022.10.20 07:49:27.045566 [ 28 ] {} <Debug> default.table_test4 (ReplicatedMergeTreeCleanupThread): Removed 38 old log entries: log-0000000680 - log-0000000717
2022.10.20 07:49:27.046377 [ 28 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeCleanupThread): Checking 9 blocks (8 are not cached) to clear old ones from ZooKeeper.
2022.10.20 07:49:27.087522 [ 28 ] {} <Debug> default.table_test4 (ReplicatedMergeTreeCleanupThread): Removed 16 old mutation entries: 0000000182 - 0000000197
2022.10.20 07:49:27.088381 [ 30 ] {} <Debug> default.table_test4 (ReplicatedMergeTreeQueue): Removing obsolete mutation 0000000182 from local state.
2022.10.20 07:49:27.088613 [ 30 ] {} <Debug> default.table_test4 (ReplicatedMergeTreeQueue): Removing obsolete mutation 0000000183 from local state.
2022.10.20 07:49:27.088769 [ 30 ] {} <Debug> default.table_test4 (ReplicatedMergeTreeQueue): Removing obsolete mutation 0000000184 from local state.
...
``` 

### 原理

对于同一个part, CK会将多个未处理的mutation合并进行批处理.

合并过程中, 会将多个mutation的条件语句进行合并, 形成SQL 向引擎查询 是否数据会被mutation变更. 这个形成的SQL过大, 导致了报错.

### 需检查

  - 27的system.mutations表

```
select count(1) from where is_done=0;

select table, length(command), length(block_numbers.partition_id), is_done, latest_fail_reason from system.mutations where is_done=0 order by length(command) desc limit 20;

select table, command from system.mutations where is_done=0 order by length(command) desc limit 3;
``` 

    - 是否存在过多积压的mutation (多个mutation一起执行, 条件会拼接成一个SQL, 会超过阈值)
    - 是否存在单个条件接近阈值的mutation (与其他少量未执行的mutation一起执行, 条件会拼接成一个SQL, 会超过阈值)

### 检查结果

![image2022-10-20 17:36:10.png](/assets/01KJBYXQWQNYSHP23Q9B3MDJCR/image2022-10-20%2017%3A36%3A10.png)

![image2022-10-20 17:36:21.png](/assets/01KJBYXQWQNYSHP23Q9B3MDJCR/image2022-10-20%2017%3A36%3A21.png)

![image2022-10-20 17:36:42.png](/assets/01KJBYXQWQNYSHP23Q9B3MDJCR/image2022-10-20%2017%3A36%3A42.png)

mutations 的 SQL 最大长度: 266k 字节, 未完成的mutations个数 28k 个, 拼接SQL的语法元素 会超过 限制.

### 后续处理 - 如何缓和状况

  - 原理: 临时降低问题CK的配置 prefer_fetch_merged_part_time_threshold 和 prefer_fetch_merged_part_size_threshold, 使得CK通过复制获取 mutations后, 更倾向于直接从源CK获取part数据, 跳过mutation变更阶段
    - 降低这两项配置, 会使得该CK的网络流量和IO增大一段时间, 直到所有mutation涉及的part被同步
  - 测试: 
    - 使用 "模拟报错" 的场景, 确认 CK-A 日志中正在报错 "AST is too long"
    - 确认 CK-A 和 CK-B 的时间是同步的

    - 在CK-A上, 调整相关表的配置: 

```
alter table table_test4 modify setting prefer_fetch_merged_part_size_threshold=1;
alter table table_test4 modify setting prefer_fetch_merged_part_time_threshold=1;
``` 
    - 检查CK-A的日志, 可以看到part复制 和 mutation被跳过的日志: 

```
//调整配置
2022.10.22 03:55:25.711087 [ 61 ] {} <Debug> executeQuery: (from 127.0.0.1:37996) alter table table_test4 modify setting prefer_fetch_merged_part_time_threshold=1
...
2022.10.22 03:55:25.723044 [ 61 ] {} <Debug> executeQuery: (from 127.0.0.1:37996) alter table table_test4 modify setting prefer_fetch_merged_part_size_threshold=1
...
 
2022.10.22 03:55:26.294211 [ 10 ] {} <Trace> default.table_test4: Executing log entry to mutate part 201901_307_328_1_406 to 201901_307_328_1_409
//part复制
2022.10.22 03:55:26.296103 [ 10 ] {} <Debug> default.table_test4: Prefer to fetch 201901_307_328_1_409 from replica 60-163
2022.10.22 03:55:26.297496 [ 10 ] {} <Debug> default.table_test4: Fetching part 201901_307_328_1_409 from /clickhouse/tables/05-02/table_test4/replicas/60-163
2022.10.22 03:55:26.299142 [ 10 ] {} <Trace> ReadWriteBufferFromHTTP: Sending request to http://10.186.60.163:9009/?endpoint=DataPartsExchange%3A%2Fclickhouse%2Ftables%2F05-02%2Ftable_test4%2Freplicas%2F60-163&part=201901_307_328_1_409&client_protocol_version=1&compress=false
2022.10.22 03:55:26.303776 [ 10 ] {} <Debug> DiskLocal: Reserving 1.00 MiB on disk `default`, having unreserved 46.63 GiB.
2022.10.22 03:55:26.307465 [ 10 ] {} <Trace> default.table_test4: Renaming temporary part tmp_fetch_201901_307_328_1_409 to 201901_307_328_1_409.
2022.10.22 03:55:26.312314 [ 10 ] {} <Debug> default.table_test4: Part 201901_307_328_1_406 is rendered obsolete by fetching part 201901_307_328_1_409
2022.10.22 03:55:26.312430 [ 10 ] {} <Debug> default.table_test4: Fetched part 201901_307_328_1_409 from /clickhouse/tables/05-02/table_test4/replicas/60-163
2022.10.22 03:55:26.314956 [ 10 ] {} <Trace> default.table_test4: Executing log entry to mutate part 201904_266_266_0_339 to 201904_266_266_0_347
2022.10.22 03:55:26.316562 [ 10 ] {} <Debug> default.table_test4: Prefer to fetch 201904_266_266_0_347 from replica 60-163
2022.10.22 03:55:26.318031 [ 10 ] {} <Debug> default.table_test4: Fetching part 201904_266_266_0_347 from /clickhouse/tables/05-02/table_test4/replicas/60-163
2022.10.22 03:55:26.318744 [ 10 ] {} <Trace> default.table_test4: Found local part 201904_266_266_0_339 with the same checksums as 201904_266_266_0_347
2022.10.22 03:55:26.319017 [ 10 ] {} <Debug> DiskLocal: Reserving 1.00 MiB on disk `default`, having unreserved 46.63 GiB.
2022.10.22 03:55:26.319402 [ 10 ] {} <Debug> default.table_test4: Cloning part /opt/clickhouse-home/data/data/default/table_test4/201904_266_266_0_339/ to /opt/clickhouse-home/data/data/default/table_test4/tmp_clone_201904_266_266_0_347
2022.10.22 03:55:26.323282 [ 10 ] {} <Trace> default.table_test4: Renaming temporary part tmp_clone_201904_266_266_0_347 to 201904_266_266_0_347.
2022.10.22 03:55:26.327973 [ 10 ] {} <Debug> default.table_test4: Part 201904_266_266_0_339 is rendered obsolete by fetching part 201904_266_266_0_347
2022.10.22 03:55:26.328141 [ 10 ] {} <Debug> default.table_test4: Cloned part 201904_266_266_0_347 from 201904_266_266_0_339
 
//跳过mutation 
2022.10.22 03:55:26.331015 [ 34 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Will check if mutation 0000000362 is done
2022.10.22 03:55:26.337646 [ 34 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Mutation 0000000362 is done
2022.10.22 03:55:27.338357 [ 41 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Marking mutation 0000000356 done because it is <= mutation_pointer (0000000362)
2022.10.22 03:55:27.338418 [ 41 ] {} <Information> default.table_test4 (ReplicatedMergeTreeQueue): Seems like we jumped over mutation 0000000356 when downloaded part with bigger
mutation number. It's OK, tasks for rest parts will be skipped, but probably a lot of mutations were executed concurrently on different replicas.
2022.10.22 03:55:27.338488 [ 41 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Marking mutation 0000000357 done because it is <= mutation_pointer (0000000362)
2022.10.22 03:55:27.338549 [ 41 ] {} <Information> default.table_test4 (ReplicatedMergeTreeQueue): Seems like we jumped over mutation 0000000357 when downloaded part with bigger
mutation number. It's OK, tasks for rest parts will be skipped, but probably a lot of mutations were executed concurrently on different replicas.
2022.10.22 03:55:27.338609 [ 41 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Marking mutation 0000000358 done because it is <= mutation_pointer (0000000362)
2022.10.22 03:55:27.338669 [ 41 ] {} <Information> default.table_test4 (ReplicatedMergeTreeQueue): Seems like we jumped over mutation 0000000358 when downloaded part with bigger
mutation number. It's OK, tasks for rest parts will be skipped, but probably a lot of mutations were executed concurrently on different replicas.
2022.10.22 03:55:27.338730 [ 41 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Marking mutation 0000000359 done because it is <= mutation_pointer (0000000362)
2022.10.22 03:55:27.338790 [ 41 ] {} <Information> default.table_test4 (ReplicatedMergeTreeQueue): Seems like we jumped over mutation 0000000359 when downloaded part with bigger
mutation number. It's OK, tasks for rest parts will be skipped, but probably a lot of mutations were executed concurrently on different replicas.
2022.10.22 03:55:27.338850 [ 41 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Marking mutation 0000000360 done because it is <= mutation_pointer (0000000362)
2022.10.22 03:55:27.338910 [ 41 ] {} <Information> default.table_test4 (ReplicatedMergeTreeQueue): Seems like we jumped over mutation 0000000360 when downloaded part with bigger
mutation number. It's OK, tasks for rest parts will be skipped, but probably a lot of mutations were executed concurrently on different replicas.
2022.10.22 03:55:27.339034 [ 41 ] {} <Trace> default.table_test4 (ReplicatedMergeTreeQueue): Marking mutation 0000000361 done because it is <= mutation_pointer (0000000362)
2022.10.22 03:55:27.339095 [ 41 ] {} <Information> default.table_test4 (ReplicatedMergeTreeQueue): Seems like we jumped over mutation 0000000361 when downloaded part with bigger
mutation number. It's OK, tasks for rest parts will be skipped, but probably a lot of mutations were executed concurrently on different replicas.
``` 
    - 检查system.mutations表 中所有项都逐渐完成: 

```
mysql> select * from system.mutations where is_done=0;
Empty set (0.02 sec)
Read 100 rows, 81.69 KiB in 0.025 sec., 3987 rows/sec., 3.18 MiB/sec.
``` 
    - 恢复表默认配置

```
alter table table_test4 modify setting prefer_fetch_merged_part_size_threshold=10737418240;
alter table table_test4 modify setting prefer_fetch_merged_part_time_threshold=3600;
``` 

### 后续处理 - 运维建议

  - 使用 alter table 修改数据时, 单个修改SQL 不要太长. 故障场景中使用了大量的IN.
  - 使用 alter table 修改数据时, 一批次SQL数量 不要太多, 跑批时将SQL间增加延迟

# 26 日志分析 - Too many parts 报错

~~检查26日志中, Too many parts报错 来自于哪台CK, 例如如下报错来自于: 10.214.113.17 CK~~

修正 (对IP地址理解有误): 报错意为SQL来自于哪个客户端 (可能是其他CK节点, 或真实客户端). 报错为本节点的报错: Too many parts

```
2022.07.16 10:08:35.962685 [ 494 ] {580482e1-f78f-4cf9-a923-bc847b2848b3} <Error> executeQuery: Code: 252, e.displayText() = DB::Exception: Too many parts (600). Merges are processing significantly slower than inserts. (version 20.1.8.41) (from 10.214.113.17:36307) (in query: insert into CK_BLACK_FILE (MD5,SOURCE,CREATE_TIME,TYPE,CONFIDENCE,LEVEL,WEB_LABEL) values), Stack trace (when copying this message, always include the lines below):

0. 0xb2087bc Poco::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/stix/clickhouse/bin/clickhouse
1. 0x4d8e3c9 DB::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/stix/clickhouse/bin/clickhouse
2. 0x888acfe DB::MergeTreeData::throwInsertIfNeeded() const  in /data/stix/clickhouse/bin/clickhouse
3. 0x8d82f18 DB::PushingToViewsBlockOutputStream::writePrefix()  in /data/stix/clickhouse/bin/clickhouse
4. 0x884cbe1 DB::DistributedBlockOutputStream::writeToLocal(DB::Block const&, unsigned long)  in /data/stix/clickhouse/bin/clickhouse
5. 0x88513af DB::DistributedBlockOutputStream::writeAsyncImpl(DB::Block const&, unsigned long)  in /data/stix/clickhouse/bin/clickhouse
6. 0x885195d DB::DistributedBlockOutputStream::writeAsync(DB::Block const&)  in /data/stix/clickhouse/bin/clickhouse
7. 0x8851db5 DB::DistributedBlockOutputStream::write(DB::Block const&)  in /data/stix/clickhouse/bin/clickhouse
8. 0x8d83f4e DB::PushingToViewsBlockOutputStream::write(DB::Block const&)  in /data/stix/clickhouse/bin/clickhouse
9. 0x8d9a330 DB::SquashingBlockOutputStream::writeSuffix()  in /data/stix/clickhouse/bin/clickhouse
10. 0x4e09d02 DB::TCPHandler::processInsertQuery(DB::Settings const&)  in /data/stix/clickhouse/bin/clickhouse
11. 0x4e0b2bb DB::TCPHandler::runImpl()  in /data/stix/clickhouse/bin/clickhouse
12. 0x4e0b769 DB::TCPHandler::run()  in /data/stix/clickhouse/bin/clickhouse
13. 0x94f4177 Poco::Net::TCPServerConnection::start()  in /data/stix/clickhouse/bin/clickhouse
14. 0x94f455b Poco::Net::TCPServerDispatcher::run()  in /data/stix/clickhouse/bin/clickhouse
15. 0xb27cb3f Poco::PooledThread::run()  in /data/stix/clickhouse/bin/clickhouse
16. 0xb279557 Poco::ThreadImpl::runnableEntry(void*)  in /data/stix/clickhouse/bin/clickhouse
17. 0xb27b526 ?  in /data/stix/clickhouse/bin/clickhouse
18. 0x7dd5 start_thread  in /usr/lib64/libpthread-2.17.so
19. 0xfeb3d __clone  in /usr/lib64/libc-2.17.so
``` 

检查报错涉及到的IP, 一共涉及如下IP: 

10.217.15.11

10.214.113.17

10.214.113.18

检查各IP报错的时间: 

来自 10.217.15.11: 

  - CK_BLACK_IP 表
    - 2022.07.27 12:42:33 -> 2022.07.27 15:50:25

来自 10.214.113.17: 

  - CK_BLACK_FILE 表
    - 2022.07.14 10:06:37 -> 2022.07.14 10:06:39
    - 2022.07.15 10:07:53 -> 2022.07.15 10:07:56
    - 2022.07.16 10:08:25 -> 2022.07.16 10:08:38
    - 2022.07.17 10:09:11 -> 2022.07.17 10:09:18
    - 2022.07.18 10:09:51 -> 2022.07.18 10:09:57
    - 2022.07.19 10:10:40 -> 2022.07.19 10:10:47
    - 2022.07.20 10:11:17 -> 2022.07.20 10:11:24
    - 2022.07.21 10:11:57 -> 2022.07.21 10:12:03
    - 2022.07.22 10:12:46 -> 2022.07.22 10:12:52
    - 2022.07.23 10:09:24 -> 2022.07.23 10:09:31
    - 2022.07.24 10:25:42 -> 2022.07.24 10:25:49
    - 2022.07.25 10:10:51 -> 2022.07.25 10:10:58
    - 2022.07.26 10:11:31 -> 2022.07.26 10:11:38
    - 2022.07.27 10:12:09 -> 2022.07.27 10:12:15
  - CK_BLACK_IP 表 
    - 2022.07.27 10:40:59 -> 2022.07.27 10:41:20
    - 2022.07.27 12:28:23 -> 2022.07.27 16:19:25

来自 10.214.113.18:

  - CK_BLACK_FILE 表
    - 2022.07.25 10:15:42 -> 2022.07.25 10:15:49
    - 2022.07.26 10:16:13 -> 2022.07.26 10:16:20
    - 2022.07.27 10:16:51 -> 2022.07.27 10:16:57

对报错现象进行梳理: 

CK_BLACK_FILE 表

  - 每天10点后, 会进行批量操作, 导致Too many parts报错. 现象持续不到半小时消失
  - 10.214.113.17 压力源会比较大, 每天10点的定时任务, 大量由 10.214.113.17 发出, 少量由 10.214.113.18 发出

CK_BLACK_IP 表

  - 压力集中于 2022.07.27 12点 -> 16点, 压力来自于 
    - 10.214.113.17, 日志中显示其是通过tcp协议连接到CK, 压力频次较高
    - 10.217.15.11, 日志中显示其是通过mysql协议连接到CK, 压力频次较低
  - 故障前后日志, 未见明显的运维操作, 认为是CK完成了Merge并自愈

建议: 

  - 检查Insert流量监控, 查看 2022.07.27 12点 -> 16点 和 每天10点的定时任务 的插入流量是否明显偏高
  - 在报错当时, 保留 system.merges 的查询结果, 判断常见的Merge慢问题: 
    - 未完成的Merge, 是否是大量mutation (mutation过多会引发part生产过多)
    - 根据 rows_read/rows_write 检查 是否有超大的merge正在进行
    - 检查进行时间长的merge线程 (根据thread_id列), 获取该线程的CPU火焰图, 或者堆栈抽样, 检查该Merge线程的工作负载

# 26 日志分析 - 其他报错

2022.07.20 14:56:38.464650 和 2022.07.20 15:00:38.573316 出现两次报错: 

```
2022.07.20 14:56:38.464650 [ 27 ] {} <Error> default.CK_BLACK_IP_LOCAL: DB::StorageReplicatedMergeTree::queueTask()::<lambda(DB::StorageReplicatedMergeTree::LogEntryPtr&)>: Code: 235, e.displayText() = DB::Exception: Part 1e40e7a5b47cc944fe40ed3d57df0123_8290538_8290546_1 (state Committed) already exists, Stack trace (when copying this message, always include the lines below):

0. 0xb2087bc Poco::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/stix/clickhouse/bin/clickhouse
1. 0x4d8e3c9 DB::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/stix/clickhouse/bin/clickhouse
2. 0x8893bc3 DB::MergeTreeData::renameTempPartAndReplace(std::__1::shared_ptr<DB::MergeTreeDataPart>&, SimpleIncrement*, DB::MergeTreeData::Transaction*, std::__1::unique_lock<std::__1::mutex>&, std::__1::vector<std::__1::shared_ptr<DB::MergeTreeDataPart const>, std::__1::allocator<std::__1::shared_ptr<DB::MergeTreeDataPart const> > >*)  in /data/stix/clickhouse/bin/clickhouse
3. 0x8894771 DB::MergeTreeData::renameTempPartAndReplace(std::__1::shared_ptr<DB::MergeTreeDataPart>&, SimpleIncrement*, DB::MergeTreeData::Transaction*)  in /data/stix/clickhouse/bin/clickhouse
4. 0x880d550 DB::StorageReplicatedMergeTree::fetchPart(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, bool, unsigned long)  in /data/stix/clickhouse/bin/clickhouse
5. 0x88101ad DB::StorageReplicatedMergeTree::executeFetch(DB::ReplicatedMergeTreeLogEntry&)  in /data/stix/clickhouse/bin/clickhouse
6. 0x8811a4f DB::StorageReplicatedMergeTree::executeLogEntry(DB::ReplicatedMergeTreeLogEntry&)  in /data/stix/clickhouse/bin/clickhouse
7. 0x88129dc ?  in /data/stix/clickhouse/bin/clickhouse
8. 0x8990bbe DB::ReplicatedMergeTreeQueue::processEntry(std::__1::function<std::__1::shared_ptr<zkutil::ZooKeeper> ()>, std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&, std::__1::function<bool (std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&)>)  in /data/stix/clickhouse/bin/clickhouse
9. 0x87d255f DB::StorageReplicatedMergeTree::queueTask()  in /data/stix/clickhouse/bin/clickhouse
10. 0x885778d DB::BackgroundProcessingPool::threadFunction()  in /data/stix/clickhouse/bin/clickhouse
11. 0x885819c ?  in /data/stix/clickhouse/bin/clickhouse
12. 0x4dc7eca ThreadPoolImpl<std::__1::thread>::worker(std::__1::__list_iterator<std::__1::thread, void*>)  in /data/stix/clickhouse/bin/clickhouse
13. 0x4dc69dc ?  in /data/stix/clickhouse/bin/clickhouse
14. 0x7dd5 start_thread  in /usr/lib64/libpthread-2.17.so
15. 0xfeb3d __clone  in /usr/lib64/libc-2.17.so
 (version 20.1.8.41)
 
 
 
2022.07.20 15:00:38.573316 [ 30 ] {} <Error> default.CK_BLACK_IP_LOCAL: DB::StorageReplicatedMergeTree::queueTask()::<lambda(DB::StorageReplicatedMergeTree::LogEntryPtr&)>: Code: 235, e.displayText() = DB::Exception: Part 1e40e7a5b47cc944fe40ed3d57df0123_8294268_8294273_1 (state Committed) already exists, Stack trace (when copying this message, always include the lines below):

0. 0xb2087bc Poco::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/stix/clickhouse/bin/clickhouse
1. 0x4d8e3c9 DB::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/stix/clickhouse/bin/clickhouse
2. 0x8893bc3 DB::MergeTreeData::renameTempPartAndReplace(std::__1::shared_ptr<DB::MergeTreeDataPart>&, SimpleIncrement*, DB::MergeTreeData::Transaction*, std::__1::unique_lock<std::__1::mutex>&, std::__1::vector<std::__1::shared_ptr<DB::MergeTreeDataPart const>, std::__1::allocator<std::__1::shared_ptr<DB::MergeTreeDataPart const> > >*)  in /data/stix/clickhouse/bin/clickhouse
3. 0x8894771 DB::MergeTreeData::renameTempPartAndReplace(std::__1::shared_ptr<DB::MergeTreeDataPart>&, SimpleIncrement*, DB::MergeTreeData::Transaction*)  in /data/stix/clickhouse/bin/clickhouse
4. 0x880d550 DB::StorageReplicatedMergeTree::fetchPart(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, bool, unsigned long)  in /data/stix/clickhouse/bin/clickhouse
5. 0x88101ad DB::StorageReplicatedMergeTree::executeFetch(DB::ReplicatedMergeTreeLogEntry&)  in /data/stix/clickhouse/bin/clickhouse
6. 0x8811a4f DB::StorageReplicatedMergeTree::executeLogEntry(DB::ReplicatedMergeTreeLogEntry&)  in /data/stix/clickhouse/bin/clickhouse
7. 0x88129dc ?  in /data/stix/clickhouse/bin/clickhouse
8. 0x8990bbe DB::ReplicatedMergeTreeQueue::processEntry(std::__1::function<std::__1::shared_ptr<zkutil::ZooKeeper> ()>, std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&, std::__1::function<bool (std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&)>)  in /data/stix/clickhouse/bin/clickhouse
9. 0x87d255f DB::StorageReplicatedMergeTree::queueTask()  in /data/stix/clickhouse/bin/clickhouse
10. 0x885778d DB::BackgroundProcessingPool::threadFunction()  in /data/stix/clickhouse/bin/clickhouse
11. 0x885819c ?  in /data/stix/clickhouse/bin/clickhouse
12. 0x4dc7eca ThreadPoolImpl<std::__1::thread>::worker(std::__1::__list_iterator<std::__1::thread, void*>)  in /data/stix/clickhouse/bin/clickhouse
13. 0x4dc69dc ?  in /data/stix/clickhouse/bin/clickhouse
14. 0x7dd5 start_thread  in /usr/lib64/libpthread-2.17.so
15. 0xfeb3d __clone  in /usr/lib64/libc-2.17.so
 (version 20.1.8.41)
``` 

错误分析:

报错是从源表复制过来的Part是本地已存在的Part, 后台线程会在10min之内重启, 并 继续复制后面的数据 (LogEntry). 

后续日志中并没有关于 CK_BLACK_IP_LOCAL的复制报错, 报错并不会影响复制进程. 由于是part重复, 也不会影响数据完整性
