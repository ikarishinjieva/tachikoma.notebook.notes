---
title: 20220915 - 光大clickhouse诊断
confluence_page_id: 1933819
created_at: 2022-09-15T10:13:16+00:00
updated_at: 2022-10-20T02:56:16+00:00
---

<https://support.actionsky.com/service_desk/browse/BEIJ-2780>

# 现象

clickhouse入库压力大，开始报错提示：Too many parts(600)

# 日志分析

日志截取连续段: 

```
2022.09.02 10:24:06.109978 [ 7 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (MergerMutator): Unexpected number of parts removed when adding a33d7c3a8f7cb805dd035209c5a83de9_3819331_3821840_551_3821846: 0 instead of 2
2022.09.02 10:27:40.179179 [ 88 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889
2022.09.02 10:27:40.179407 [ 88 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889.
2022.09.02 10:27:40.179999 [ 88 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Found parts with the same min block and with the same max block as the missing part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889. Hoping that it will eventually appear as a result of a merge.
2022.09.02 10:27:40.200717 [ 86 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889
2022.09.02 10:27:40.200956 [ 86 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889.
2022.09.02 10:27:40.201597 [ 86 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Found parts with the same min block and with the same max block as the missing part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889. Hoping that it will eventually appear as a result of a merge.
2022.09.02 10:27:40.309610 [ 56 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889
2022.09.02 10:27:40.309847 [ 56 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889.
2022.09.02 10:27:40.310459 [ 56 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Found parts with the same min block and with the same max block as the missing part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889. Hoping that it will eventually appear as a result of a merge.
2022.09.02 10:27:40.398033 [ 17 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889
2022.09.02 10:27:40.398173 [ 17 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889.
2022.09.02 10:27:40.398786 [ 17 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Found parts with the same min block and with the same max block as the missing part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889. Hoping that it will eventually appear as a result of a merge.
2022.09.02 10:27:40.427697 [ 11 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889
2022.09.02 10:27:40.427838 [ 11 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889.
2022.09.02 10:27:40.428479 [ 11 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Found parts with the same min block and with the same max block as the missing part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889. Hoping that it will eventually appear as a result of a merge.
2022.09.02 10:27:40.491497 [ 54 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889
2022.09.02 10:27:40.491644 [ 54 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889.
2022.09.02 10:27:40.491992 [ 54 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Found parts with the same min block and with the same max block as the missing part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889. Hoping that it will eventually appear as a result of a merge.
2022.09.02 10:27:41.825559 [ 30 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL: Tried to add obsolete part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821887 covered by a33d7c3a8f7cb805dd035209c5a83de9_3819331_3821891_559_3821890 (state Committed)
2022.09.02 10:30:05.272353 [ 69 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part a33d7c3a8f7cb805dd035209c5a83de9_3819331_3821927_567_3821918
2022.09.02 10:30:05.272532 [ 69 ] {} <Warning> default.CK_BLACK_PHONE_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering a33d7c3a8f7cb805dd035209c5a83de9_3819331_3821927_567_3821918.
``` 
    
    
    知识

  - 对part命名的解释: a33d7c3a8f7cb805dd035209c5a83de9_3819331_3821891_559_3821890
    - ![image2022-9-15 17:1:36.png](/assets/01KJBYXQPQ1SX1N9ZQFRYPFKGG/image2022-9-15%2017%3A1%3A36.png)
    - 注意: mutation的版本号 跟 block号 接近
  - ReplicatedMergeTreePartCheckThread的作用: 检查本地分区是否损坏
    - 检查本地分区和zk中的分区信息的不一致
      - 将 本地表的 column 和zk中表的column信息比较
      - 将 本地表的 元信息 和zk中表的 元信息 比较
      - 将 本地表的 数据checksum 和zk中表的 数据checksum 比较
  - ClickHouse复制表同步机制浅析: <https://www.jianshu.com/p/11ce7a327c62>

  - bug逻辑

    - ReplicatedMergeTreePartCheckThread 要检查一个分区 a-b 是否存在 (是什么触发了检查? )
      - 在本地未发现 a-b (ReplicatedMergeTreePartCheckThread::checkPart)
      - 在其他节点搜索 a-b (searchForMissingPart)
        - 没找到 a-b
        - 找到了 a-x, y-b
        - 报错类似: 

```
Found parts with the same min block and with the same max block as the missing part a33d7c3a8f7cb805dd035209c5a83de9_3821886_3821886_0_3821889. Hoping that it will eventually appear as a result of a merge
```

        - 寄希望于 其他节点能将 a-x, y-b进行合并, 生成 a-b

# 现象2

机器27的日志: 

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

堆栈显示:

  - 在复制过程中, 执行回访的线程, 正在执行 Mutation (数据变更)
  - 数据变更 要检查 isStorageTouchedByMutations (数据是否真的被变更)
    - 其中要执行检查SQL, 在拼接这个检查SQL时, 由于传入的MutationCommand (数据变更指令) 过多, 导致SQL过大, 超过阈值
    - MutationCommand 为什么会过多  

      - 来源于: queue.getMutationCommands
      - 复现步骤: 
        - 搭建CK复制
        - 插入数据 
        - debug断点打在 DB::isStorageTouchedByMutations, 在另一台CK上同时下多个 alter table
        - 可观察到 一次需要处理的MutationCommand增多
        - 便捷命令: 

```
CREATE TABLE default.table_test4 (`EventDate` DateTime, `CounterID` UInt32, `UserID` UInt32, `AAA` UInt32) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/table_test4', '{replica}') PARTITION BY toYYYYMM(EventDate) ORDER BY (CounterID, EventDate, intHash32(UserID)) SAMPLE BY intHash32(UserID) SETTINGS index_granularity = 8192
 
insert into table_test4 values('2019-01-01 00:00:04',0,0,0);
insert into table_test4 values('2019-01-02 00:00:04',0,0,0);
insert into table_test4 values('2019-01-03 00:00:04',0,0,0);
insert into table_test4 values('2019-01-04 00:00:04',0,0,0);
insert into table_test4 values('2019-01-05 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);
insert into table_test4 values('2019-01-06 00:00:04',0,0,0);

insert into table_test4 values('2019-02-01 00:00:04',0,0,0);
insert into table_test4 values('2019-03-01 00:00:04',0,0,0);
insert into table_test4 values('2019-04-01 00:00:04',0,0,0);
insert into table_test4 values('2019-05-01 00:00:04',0,0,0);
insert into table_test4 values('2019-06-01 00:00:04',0,0,0);
insert into table_test4 values('2019-07-01 00:00:04',0,0,0);
insert into table_test4 values('2019-08-01 00:00:04',0,0,0);
insert into table_test4 values('2019-09-01 00:00:04',0,0,0);
insert into table_test4 values('2019-10-01 00:00:04',0,0,0);

alter table table_test4 update AAA =1 where EventDate = '2019-01-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-01-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-01-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-01-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-01-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-01-01 00:00:04';

alter table table_test4 update AAA =1 where EventDate = '2019-02-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-03-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-04-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-05-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-06-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-07-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-08-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-09-01 00:00:04';
alter table table_test4 update AAA =1 where EventDate = '2019-10-01 00:00:04';

```

# 常用命令

```
/usr/share/zookeeper/bin/zkServer.sh
 
/etc/zookeeper/conf/zoo.cfg
 
/usr/share/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181
 
/opt/ClickHouse/build/dbms/programs# nohup ./clickhouse server start -C /opt/clickhouse-home/config.xml &
 
~/opt/mysql/5.7.25/bin/mysql -h127.0.0.1 -P9004 -u default
 
CREATE TABLE table_test4
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/table_test4', '{replica}')
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
```
