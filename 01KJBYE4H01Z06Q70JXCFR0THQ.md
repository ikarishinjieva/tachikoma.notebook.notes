---
title: 20210811 - 天津滨银汽车Clickhouse问题咨询
confluence_page_id: 1343612
created_at: 2021-08-11T14:43:37+00:00
updated_at: 2021-08-19T03:14:22+00:00
---

# 原始问题

[clickhouse查询问题收集.docx](/assets/01KJBYE4H01Z06Q70JXCFR0THQ/clickhouse%E6%9F%A5%E8%AF%A2%E9%97%AE%E9%A2%98%E6%94%B6%E9%9B%86.docx)

# 实验

架设Clickhouse 21.3.2.5

架设到MySQL的复制: 

```
CREATE DATABASE mysql ENGINE = MaterializeMySQL('127.0.0.1:21526', 'clickhouse', 'aaa', 'aaa');
``` 

在MySQL端创建表: 

```
CREATE TABLE `tb_intermediate` (
  `id` varchar(20) COLLATE utf8mb4_bin NOT NULL,
  `f_vch_number` varchar(20) COLLATE utf8mb4_bin DEFAULT NULL,
  `f_flag` varchar(20) COLLATE utf8mb4_bin DEFAULT NULL,
  `f_amount` decimal(12,2) DEFAULT NULL,
  `f_trans_date` datetime DEFAULT NULL,
  `f_account_no` varchar(30) COLLATE utf8mb4_bin DEFAULT NULL,
  `f_account_cd` varchar(30) COLLATE utf8mb4_bin DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `ind_flag` (`f_flag`),
  KEY `ind_fvch` (`f_vch_number`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
 
 
insert into tb_intermediate values('1','1','1',100,'2021-08-06 00:00:00','10020301','271375440926');
insert into tb_intermediate values('2','2','2',100,'2021-08-06 01:00:00','10020301','271375440926');
insert into tb_intermediate values('3','3','3',100,'2021-08-06 02:00:00','10020301','271375440926');
insert into tb_intermediate values('4','4','4',100,'2021-08-06 03:00:00','10020301','271375440926');
insert into tb_intermediate values('5','5','5',100,'2021-08-06 04:00:00','10020301','271375440926');
insert into tb_intermediate values('6','6','6',100,'2021-08-06 05:00:00','10020301','271375440926');
insert into tb_intermediate values('7','7','7',100,'2021-08-06 06:00:00','10020301','271375440926');
insert into tb_intermediate values('8','8','8',100,'2021-08-06 07:00:00','10020301','271375440926');
insert into tb_intermediate values('9','9','9',100,'2021-08-06 08:00:00','10020301','271375440926');
insert into tb_intermediate values('10','10','10',100,'2021-08-06 09:00:00','10020301','271375440926');
insert into tb_intermediate values('11','11','11',100,'2021-08-06 10:00:00','10020301','271375440926');
insert into tb_intermediate values('12','12','12',100,'2021-08-06 11:00:00','10020301','271375440926');
insert into tb_intermediate values('13','13','13',100,'2021-08-06 12:00:00','10020301','271375440926');
insert into tb_intermediate values('14','14','14',100,'2021-08-06 13:00:00','10020301','271375440926');
insert into tb_intermediate values('15','15','15',100,'2021-08-06 14:00:00','10020301','271375440926');
``` 

# 问题1

执行查询: 

```
select  sum(f_amount) from  tb_intermediate ti  
 where    
 f_trans_date >='2021-08-06 00:00:00' 
 and f_trans_date < '2021-08-07 00:00:00' 
 and f_account_no ='10020301' 
 and f_account_cd ='271375440926';
``` 

报错与用户现场相同: 

```
SELECT sum(f_amount)
FROM tb_intermediate AS ti
WHERE (f_trans_date >= '2021-08-06 00:00:00') AND (f_trans_date < '2021-08-07 00:00:00') AND (f_account_cd = '10020301') AND (f_account_no = '271375440926')

Query id: c0effc28-194e-4217-ad8c-d93d5b04d73e

[ubuntu] 2021.08.11 14:50:51.981786 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Debug> executeQuery: (from [::ffff:127.0.0.1]:42352, using production parser) select sum(f_amount) from tb_intermediate ti where f_trans_date >='2021-08-06 00:00:00' and f_trans_date < '2021-08-07 00:00:00' and f_account_cd ='10020301' and f_account_no ='271375440926';
[ubuntu] 2021.08.11 14:50:51.982793 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Debug> InterpreterSelectQuery: MergeTreeWhereOptimizer: condition "(f_trans_date >= '2021-08-06 00:00:00') AND (f_trans_date < '2021-08-07 00:00:00')" moved to PREWHERE
[ubuntu] 2021.08.11 14:50:51.983615 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Trace> ContextAccess (default): Access granted: SELECT(f_amount, f_trans_date, f_account_no, f_account_cd) ON mysql.tb_intermediate
[ubuntu] 2021.08.11 14:50:51.983878 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Debug> mysql.tb_intermediate (ad5bd3b2-f8d4-4ac4-aa50-5155c6f9aa0e) (SelectExecutor): Key condition: unknown, unknown, and, unknown, unknown, and, and, unknown, unknown, and, and
[ubuntu] 2021.08.11 14:50:51.984291 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Debug> mysql.tb_intermediate (ad5bd3b2-f8d4-4ac4-aa50-5155c6f9aa0e) (SelectExecutor): Selected 2/2 parts by partition key, 2 parts by primary key, 2/4 marks by primary key, 2 marks to read from 2 ranges
[ubuntu] 2021.08.11 14:50:51.984420 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Trace> MergeTreeSelectProcessor: Reading 1 ranges from part all_1_1_0, approx. 14 rows starting from 0
[ubuntu] 2021.08.11 14:50:51.984501 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Trace> MergeTreeSelectProcessor: Reading 1 ranges from part all_2_2_0, approx. 1 rows starting from 0
[ubuntu] 2021.08.11 14:50:51.986810 [ 22762 ] {c0effc28-194e-4217-ad8c-d93d5b04d73e} <Error> executeQuery: Code: 47, e.displayText() = DB::Exception: Missing columns: 'f_trans_date' while processing query: '_sign = 1, f_amount, f_trans_date, f_account_no, f_account_cd', required columns: '_sign' 'f_account_cd' 'f_amount' 'f_trans_date' 'f_account_no' '_sign' 'f_account_cd' 'f_amount' 'f_trans_date' 'f_account_no' (version 21.3.2.5 (official build)) (from [::ffff:127.0.0.1]:42352) (in query: select sum(f_amount) from tb_intermediate ti where f_trans_date >='2021-08-06 00:00:00' and f_trans_date < '2021-08-07 00:00:00' and f_account_cd ='10020301' and f_account_no ='271375440926';), Stack trace (when copying this message, always include the lines below):

0. DB::TreeRewriterResult::collectUsedColumns(std::__1::shared_ptr<DB::IAST> const&, bool) @ 0xf0c9e4e in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
1. DB::TreeRewriter::analyze(std::__1::shared_ptr<DB::IAST>&, DB::NamesAndTypesList const&, std::__1::shared_ptr<DB::IStorage const>, std::__1::shared_ptr<DB::StorageInMemoryMetadata const> const&, bool) const @ 0xf0d2c7a in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
2. DB::StorageMaterializeMySQL::read(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::shared_ptr<DB::StorageInMemoryMetadata const> const&, DB::SelectQueryInfo&, DB::Context const&, DB::QueryProcessingStage::Enum, unsigned long, unsigned int) @ 0xf374962 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
3. DB::IStorage::read(DB::QueryPlan&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::shared_ptr<DB::StorageInMemoryMetadata const> const&, DB::SelectQueryInfo&, DB::Context const&, DB::QueryProcessingStage::Enum, unsigned long, unsigned int) @ 0xf2d603f in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
4. DB::InterpreterSelectQuery::executeFetchColumns(DB::QueryProcessingStage::Enum, DB::QueryPlan&) @ 0xec827b1 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
5. DB::InterpreterSelectQuery::executeImpl(DB::QueryPlan&, std::__1::shared_ptr<DB::IBlockInputStream> const&, std::__1::optional<DB::Pipe>) @ 0xec77a3c in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
6. DB::InterpreterSelectQuery::buildQueryPlan(DB::QueryPlan&) @ 0xec768eb in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
7. DB::InterpreterSelectWithUnionQuery::buildQueryPlan(DB::QueryPlan&) @ 0xef919c3 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
8. DB::InterpreterSelectWithUnionQuery::execute() @ 0xef92b4b in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
9. DB::executeQueryImpl(char const*, char const*, DB::Context&, bool, DB::QueryProcessingStage::Enum, bool, DB::ReadBuffer*) @ 0xf12d3a2 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
10. DB::executeQuery(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, DB::Context&, bool, DB::QueryProcessingStage::Enum, bool) @ 0xf12bce3 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
11. DB::TCPHandler::runImpl() @ 0xf8b7c5d in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
12. DB::TCPHandler::run() @ 0xf8ca1c9 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
13. Poco::Net::TCPServerConnection::start() @ 0x11f7ccbf in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
14. Poco::Net::TCPServerDispatcher::run() @ 0x11f7e6d1 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
15. Poco::PooledThread::run() @ 0x120b4df9 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
16. Poco::ThreadImpl::runnableEntry(void*) @ 0x120b0c5a in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
17. start_thread @ 0x76db in /lib/x86_64-linux-gnu/libpthread-2.27.so
18. /build/glibc-S9d2JN/glibc-2.27/misc/../sysdeps/unix/sysv/linux/x86_64/clone.S:97: __clone @ 0x12171f in /usr/lib/debug/lib/x86_64-linux-gnu/libc-2.27.so

0 rows in set. Elapsed: 0.007 sec.

Received exception from server (version 21.3.2):
Code: 47. DB::Exception: Received from localhost:9000. DB::Exception: Missing columns: 'f_trans_date' while processing query: '_sign = 1, f_amount, f_trans_date, f_account_no, f_account_cd', required columns: '_sign' 'f_account_cd' 'f_amount' 'f_trans_date' 'f_account_no' '_sign' 'f_account_cd' 'f_amount' 'f_trans_date' 'f_account_no'.
``` 

找到相关issue: <https://github.com/ClickHouse/ClickHouse/issues/25794>

逻辑: MaterializedMySQL 在AST的根上新生成了一个父根, 将 sign=1 条件加在父根上. 当做解析时, clickhouse从该父根上找相关列, 由于父根是虚拟出来的, 所以找不到相关列, 报错. 

修改: clickhouse做解析时, 对于rewrite的父根, 在其子节点中查找相关列, 避免报错

# 问题2

以下SQL理应报错 (_sign没有对比符号), 但可以执行

```
localhost :) SELECT * FROM tb_intermediate AS ti　WHERE  f_flag='1'  and  _sign;

SELECT *
FROM tb_intermediate AS ti
WHERE (f_flag = '1') AND _sign

Query id: 6fedc038-8e6c-4113-a93d-bf25547107ab

┌─id─┬─f_vch_number─┬─f_flag─┬─f_amount─┬────────f_trans_date─┬─f_account_no─┬─f_account_cd─┬─number─┬─str─┐
│ 1  │ 1            │ 1      │   100.00 │ 2021-08-06 00:00:00 │ 10020301     │ 271375440926 │      0 │     │
└────┴──────────────┴────────┴──────────┴─────────────────────┴──────────────┴──────────────┴────────┴─────┘

1 rows in set. Elapsed: 0.008 sec.
``` 

条件变更为 id后, 能正常报错

```
localhost :) SELECT * FROM tb_intermediate AS ti　WHERE  id='1'  and  _sign;

SELECT *
FROM tb_intermediate AS ti
WHERE (id = '1') AND _sign

Query id: 8e29dfd0-65ec-484b-9986-7b7b71fbe483

0 rows in set. Elapsed: 0.009 sec.

Received exception from server (version 21.3.2):
Code: 59. DB::Exception: Received from localhost:9000. DB::Exception: Illegal type Int8 of column for filter. Must be UInt8 or Nullable(UInt8) or Const variants of them.: While executing MergeTree.
 
``` 

去掉id后, 也能正常报错: 

```
localhost :) SELECT * FROM tb_intermediate AS ti　WHERE  _sign;

SELECT *
FROM tb_intermediate AS ti
WHERE _sign

Query id: 72c4d9e4-c94a-4553-8730-ce465e77f8be

0 rows in set. Elapsed: 0.006 sec.

Received exception from server (version 21.3.2):
Code: 59. DB::Exception: Received from localhost:9000. DB::Exception: Illegal type Int8 of column for filter. Must be UInt8 or Nullable(UInt8) or Const variants of them.: While executing MergeTree.
``` 

报错堆栈: 

```
2021.08.12 13:16:35.540902 [ 29035 ] {72c4d9e4-c94a-4553-8730-ce465e77f8be} <Error> TCPHandler: Code: 59, e.displayText() = DB::Exception: Illegal type Int8 of column for filter. Must be UInt8 or Nullable
(UInt8) or Const variants of them.: While executing MergeTree, Stack trace:

0. DB::FilterDescription::FilterDescription(DB::IColumn const&) @ 0xf2b1a21 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
1. DB::MergeTreeRangeReader::ReadResult::setFilter(COW<DB::IColumn>::immutable_ptr<DB::IColumn> const&) @ 0xf707ab2 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
2. DB::MergeTreeRangeReader::executePrewhereActionsAndFilterColumns(DB::MergeTreeRangeReader::ReadResult&) @ 0xf70abd1 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
3. DB::MergeTreeRangeReader::read(unsigned long, std::__1::deque<DB::MarkRange, std::__1::allocator<DB::MarkRange> >&) @ 0xf7089c4 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
4. DB::MergeTreeRangeReader::read(unsigned long, std::__1::deque<DB::MarkRange, std::__1::allocator<DB::MarkRange> >&) @ 0xf7087e3 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
5. DB::MergeTreeBaseSelectProcessor::readFromPartImpl() @ 0xf7021b3 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
6. DB::MergeTreeBaseSelectProcessor::readFromPart() @ 0xf702e8d in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
7. DB::MergeTreeBaseSelectProcessor::generate() @ 0xf7016ab in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
8. DB::ISource::tryGenerate() @ 0xf8f7df5 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
9. DB::ISource::work() @ 0xf8f79ea in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
10. DB::SourceWithProgress::work() @ 0xfaaa4ba in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
11. void std::__1::__function::__policy_invoker<void ()>::__call_impl<std::__1::__function::__default_alloc_func<DB::PipelineExecutor::addJob(DB::ExecutingGraph::Node*)::$_0, void ()> >(std::__1::__function::__policy_storage const*) @ 0xf931ddd in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
12. DB::PipelineExecutor::executeStepImpl(unsigned long, unsigned long, std::__1::atomic<bool>*) @ 0xf92ea01 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
13. void std::__1::__function::__policy_invoker<void ()>::__call_impl<std::__1::__function::__default_alloc_func<ThreadFromGlobalPool::ThreadFromGlobalPool<DB::PipelineExecutor::executeImpl(unsigned long)::$_4>(DB::PipelineExecutor::executeImpl(unsigned long)::$_4&&)::'lambda'(), void ()> >(std::__1::__function::__policy_storage const*) @ 0xf9335d6 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
14. ThreadPoolImpl<std::__1::thread>::worker(std::__1::__list_iterator<std::__1::thread, void*>) @ 0x864f9df in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
15. void* std::__1::__thread_proxy<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct> >, void ThreadPoolImpl<std::__1::thread>::scheduleImpl<void>(std::__1::function<void ()>, int, std::__1::optional<unsigned long>)::'lambda1'()> >(void*) @ 0x8653473 in /usr/lib/debug/.build-id/17/9bdbbf228667883dfcd900b8cd498272fc044f.debug
16. start_thread @ 0x76db in /lib/x86_64-linux-gnu/libpthread-2.27.so
17. /build/glibc-S9d2JN/glibc-2.27/misc/../sysdeps/unix/sysv/linux/x86_64/clone.S:97: __clone @ 0x12171f in /usr/lib/debug/lib/x86_64-linux-gnu/libc-2.27.so
``` 

更换SQL, 增加 toUint8, 可以正确执行:

```
localhost :) SELECT *,_sign FROM tb_intermediate AS ti　WHERE  toUInt8(_sign);

SELECT
    *,
    _sign
FROM tb_intermediate AS ti
WHERE toUInt8(_sign)

Query id: 1481d63f-c75c-4c53-af21-a63b530c8a22

┌─id─┬─f_vch_number─┬─f_flag─┬─f_amount─┬────────f_trans_date─┬─f_account_no─┬─f_account_cd─┬─number─┬─str─┬─_sign─┐
│ 13 │ 13           │ 13     │   200.00 │ 2021-08-06 12:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 15 │ 15           │ 15     │   200.00 │ 2021-08-06 14:00:00 │ 10020301     │ 271375440926 │      0 │     │    -1 │
└────┴──────────────┴────────┴──────────┴─────────────────────┴──────────────┴──────────────┴────────┴─────┴───────┘
┌─id─┬─f_vch_number─┬─f_flag─┬─f_amount─┬────────f_trans_date─┬─f_account_no─┬─f_account_cd─┬─number─┬─str─┬─_sign─┐
│ 12 │ 12           │ 12     │   200.00 │ 2021-08-06 11:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 14 │ 14           │ 14     │   200.00 │ 2021-08-06 13:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
└────┴──────────────┴────────┴──────────┴─────────────────────┴──────────────┴──────────────┴────────┴─────┴───────┘
┌─id─┬─f_vch_number─┬─f_flag─┬─f_amount─┬────────f_trans_date─┬─f_account_no─┬─f_account_cd─┬─number─┬─str─┬─_sign─┐
│ 1  │ 1            │ 1      │   100.00 │ 2021-08-06 00:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 2  │ 2            │ 2      │   100.00 │ 2021-08-06 01:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 3  │ 3            │ 3      │   100.00 │ 2021-08-06 02:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 4  │ 4            │ 4      │   100.00 │ 2021-08-06 03:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 5  │ 5            │ 5      │   100.00 │ 2021-08-06 04:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 6  │ 6            │ 6      │   100.00 │ 2021-08-06 05:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 7  │ 7            │ 7      │   100.00 │ 2021-08-06 06:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 8  │ 8            │ 8      │   100.00 │ 2021-08-06 07:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
│ 9  │ 9            │ 9      │   100.00 │ 2021-08-06 08:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
└────┴──────────────┴────────┴──────────┴─────────────────────┴──────────────┴──────────────┴────────┴─────┴───────┘
┌─id─┬─f_vch_number─┬─f_flag─┬─f_amount─┬────────f_trans_date─┬─f_account_no─┬─f_account_cd─┬─number─┬─str─┬─_sign─┐
│ 10 │ 10           │ 10     │   100.00 │ 2021-08-06 09:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
└────┴──────────────┴────────┴──────────┴─────────────────────┴──────────────┴──────────────┴────────┴─────┴───────┘
┌─id─┬─f_vch_number─┬─f_flag─┬─f_amount─┬────────f_trans_date─┬─f_account_no─┬─f_account_cd─┬─number─┬─str─┬─_sign─┐
│ 11 │ 11           │ 11     │   200.00 │ 2021-08-06 10:00:00 │ 10020301     │ 271375440926 │      0 │     │     1 │
└────┴──────────────┴────────┴──────────┴─────────────────────┴──────────────┴──────────────┴────────┴─────┴───────┘

15 rows in set. Elapsed: 0.006 sec.

``` 

需要查找原因: 

客户版本: 21.3.10.1, 编译Debug版本

在executePrewhereActionsAndFilterColumns中, 对比 ( id='1' and _sign) 和 (f_flag='1' and _sign)的区别: 

  - id='1' and _sign
    - num_columns = 1
    - prewhere_column_name = "_sign"
    - 报错的原因是 CK认为 prewhere的表达式应该是 boolean, 以UInt8表达, 而 _sign 是Int8, 类型冲突
  - f_flag='1' and _sign
    - num_columns = 2
    - prewhere_column_name = "and(equals(f_flag, '1'), _sign)"
    - prewhere的表达式 是 and, 类型正确

需要调查 (id='1' and _sign)时, prewhere选取的列为什么只有一列

经过调试分析, where转向prewhere的方法在 MergeTreeWhereOptimizer::optimize, 进行分析: 

  - id='1' and _sign
    - 仅有一个条件_sign从where移向prewhere
    - 遍历到id时, 触发到条件 !viable, 终止优化

```
while (!where_conditions.empty())
    {
        /// Move the best condition to PREWHERE if it is viable.

        auto it = std::min_element(where_conditions.begin(), where_conditions.end());

        if (!it->viable)
            break;

        ...

        move_condition(it);
    }
```

  - f_flag='1' and _sign
    - 两个条件正常从where移向prewhere

什么是viable条件: 

```
        cond.viable =
            /// Condition depend on some column. Constant expressions are not moved.
            !cond.identifiers.empty()
            && !cannotBeMoved(node)
            /// Do not take into consideration the conditions consisting only of the first primary key column
            && !hasPrimaryKeyAtoms(node)
            /// Only table columns are considered. Not array joined columns. NOTE We're assuming that aliases was expanded.
            && isSubsetOfTableColumns(cond.identifiers)
            /// Do not move conditions involving all queried columns.
            && cond.identifiers.size() < queried_columns.size();
``` 

对于ID列, 不符合条件 !hasPrimaryKeyAtoms, id是构成PK的第一列

将prewhere优化关闭, 都可以正常进行: 

```
localhost :) SELECT * FROM tb_intermediate AS ti　WHERE  id='1'  and  _sign settings optimize_move_to_prewhere=0;

SELECT *
FROM tb_intermediate AS ti
WHERE (id = '1') AND _sign
SETTINGS optimize_move_to_prewhere = 0

Query id: b72653d9-a4e4-4dc3-a7ef-1c30cf53b016

┌─id─┬─f_vch_number─┬─f_flag─┬─f_amount─┬────────f_trans_date─┬─f_account_no─┬─f_account_cd─┬─number─┬─str─┐
│ 1  │ 1            │ 1      │   100.00 │ 2021-08-06 00:00:00 │ 10020301     │ 271375440926 │      0 │     │
└────┴──────────────┴────────┴──────────┴─────────────────────┴──────────────┴──────────────┴────────┴─────┘

1 rows in set. Elapsed: 0.056 sec.
``` 

# 问题3

用户补全了问题描述: 

  1. 如描述文档, sum确实出现了问题
  2. 而后, 用户执行了optimize table final, 之后sum恢复正常
  3. 现场已丢失, 无法追踪

目前无法复现
