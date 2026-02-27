---
title: 20221019 - clickhouse "table is in readonly mode" 诊断
confluence_page_id: 1934304
created_at: 2022-10-19T13:26:51+00:00
updated_at: 2022-10-19T14:18:53+00:00
---

# Readonly诊断

日志包: [cklog.tar.gz](/assets/01KJBYPH9KR9B4KT123DR6PJAF/cklog.tar.gz)

日志完整时间段: 2022.10.12 10:47:28.686628 - 2022.10.12 16:52:01.420973

日志中, "Table is in readonly mode" 只从 2022.10.12 10:47:28.686628 持续到 2022.10.12 15:42:30.199616

判断在 2022.10.12 15:42:30.199616 之后, 环境发生了变化 (网络/Zookeeper), 使得只读状况恢复

日志中检查到 Zookeeper Session timeout日志

```
2022.10.12 11:02:17.372624 [ 16 ] {} <Error> default.CK_ANALYSIS_HOSTVUL_LOCAL: DB::StorageReplicatedMergeTree::queueTask()::<lambda(DB::StorageReplicatedMergeTree::LogEntryPtr&)>: Code: 999, e.displayText() = Coordination::Exception: Session expired (Session expired), Stack trace (when copying this message, always include the lines below):

0. 0xb2087bc Poco::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/asap/clickhouse/bin/clickhouse
1. 0x4d8e3c9 DB::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/asap/clickhouse/bin/clickhouse
2. 0x909f8a6 ?  in /data/asap/clickhouse/bin/clickhouse
3. 0x90a015c Coordination::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/asap/clickhouse/bin/clickhouse
4. 0x90b5e73 Coordination::ZooKeeper::pushRequest(Coordination::ZooKeeper::RequestInfo&&)  in /data/asap/clickhouse/bin/clickhouse
5. 0x90b6752 Coordination::ZooKeeper::exists(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, std::__1::function<void (Coordination::ExistsResponse const&)>, std::__1::function<void (Coordination::WatchResponse const&)>)  in /data/asap/clickhouse/bin/clickhouse
6. 0x90a4abb zkutil::ZooKeeper::existsImpl(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, Coordination::Stat*, std::__1::function<void (Coordination::WatchResponse const&)>)  in /data/asap/clickhouse/bin/clickhouse
7. 0x90a4c2f zkutil::ZooKeeper::exists(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, Coordination::Stat*, std::__1::shared_ptr<Poco::Event> const&)  in /data/asap/clickhouse/bin/clickhouse
8. 0x8811ba4 DB::StorageReplicatedMergeTree::executeLogEntry(DB::ReplicatedMergeTreeLogEntry&)  in /data/asap/clickhouse/bin/clickhouse
9. 0x88129dc ?  in /data/asap/clickhouse/bin/clickhouse
10. 0x8990bbe DB::ReplicatedMergeTreeQueue::processEntry(std::__1::function<std::__1::shared_ptr<zkutil::ZooKeeper> ()>, std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&, std::__1::function<bool (std::__1::shared_ptr<DB::ReplicatedMergeTreeLogEntry>&)>)  in /data/asap/clickhouse/bin/clickhouse
11. 0x87d255f DB::StorageReplicatedMergeTree::queueTask()  in /data/asap/clickhouse/bin/clickhouse
12. 0x885778d DB::BackgroundProcessingPool::threadFunction()  in /data/asap/clickhouse/bin/clickhouse
13. 0x885819c ?  in /data/asap/clickhouse/bin/clickhouse
14. 0x4dc7eca ThreadPoolImpl<std::__1::thread>::worker(std::__1::__list_iterator<std::__1::thread, void*>)  in /data/asap/clickhouse/bin/clickhouse
15. 0x4dc69dc ?  in /data/asap/clickhouse/bin/clickhouse
16. 0x7dd5 start_thread  in /usr/lib64/libpthread-2.17.so
17. 0xfeb3d __clone  in /usr/lib64/libc-2.17.so
 (version 20.1.8.41)
``` 

呈现周期性, 每10分钟打印一次: 

![image2022-10-19 19:56:42.png](/assets/01KJBYPH9KR9B4KT123DR6PJAF/image2022-10-19%2019%3A56%3A42.png)

复合后台线程的调度逻辑: 如果后台线程出现错误, 后台线程会在 {task_sleep_seconds_when_no_work_min, task_sleep_seconds_when_no_work_max} 时间区间睡眠, 并逐步增加睡眠时间, 最大睡眠时间为 600s (task_sleep_seconds_when_no_work_max)

(代码: BackgroundProcessingPool::threadFunction)

根据clickhouse逻辑, 当ZK的会话超时 (Session Expired), 就会将表置为 只读 状态, 并在下次使用ZK时尝试重新建立连接 (代码: Context::getZooKeeper): 

![image2022-10-19 20:26:35.png](/assets/01KJBYPH9KR9B4KT123DR6PJAF/image2022-10-19%2020%3A26%3A35.png)

什么会导致对ZK的访问形成 Session Expired状态? 通常的原因: 

  1. ZK集群 整体或局部 响应慢, 或失去leader (模拟: 通过gdb拖慢ZK进程, 可以模拟出类似日志)
  2. 网络出现延迟, 使得网络包响应 超过ZK的会话超时时间 (模拟: 通过iptables将CK和ZK的网络静默, 可以模拟出类似日志)  

     1. 长时间网络问题, CK会出现如下报错日志: "All connection tries failed while connecting to ZooKeeper". 与本现象不符

故障时间段内, 还穿插出现了如下日志: 

```
2022.10.12 10:52:24.798044 [ 49 ] {} <Warning> default.CK_ANALYSIS_TRANSMONITOR_LOCAL (ReplicatedMergeTreePartCheckThread): Checking part 20221012_39430_39687_54
2022.10.12 10:52:24.798199 [ 49 ] {} <Warning> default.CK_ANALYSIS_TRANSMONITOR_LOCAL (ReplicatedMergeTreePartCheckThread): Checking if anyone has a part covering 20221012_39430_39687_54.
2022.10.12 10:52:24.799477 [ 49 ] {} <Warning> default.CK_ANALYSIS_TRANSMONITOR_LOCAL (ReplicatedMergeTreePartCheckThread): Found parts with the same min block and with the same max block as the missing part 20221012_39430_39687_54. Hoping that it will eventually appear as a result of a merge.
``` 

相关日志, 是在访问ZK获取数据后才会输出, 代码逻辑: 

![image2022-10-19 20:31:53.png](/assets/01KJBYPH9KR9B4KT123DR6PJAF/image2022-10-19%2020%3A31%3A53.png)

可以判断: 在故障时间段内, ZK集群是部分可以正常工作的, 而部分异常. ZK故障时间段后, CK会自动恢复正常

建议: 检查故障时间段, ZK集群的健康状况

# 其余问题

日志中, 反复出现如下报错: 

```
2022.10.12 10:47:29.041866 [ 399 ] {} <Error> CK_ANALYSIS_HOSTVUL.Distributed.DirectoryMonitor: Code: 252, e.displayText() = DB::Exception: Received from CK3:24902. DB::Exception: Too many partitions for single INSERT block (more than 100). The limit is controlled by 'max_partitions_per_insert_block' setting. Large number of partitions is a common misconception. It will lead to severe negative performance impact, including slow server startup, slow INSERT queries and slow SELECT queries. Recommended total number of partitions for a table is under 1000..10000. Please note, that partitioning is not intended to speed up SELECT queries (ORDER BY key is sufficient to make range queries fast). Partitions are intended for data manipulation (DROP PARTITION, etc).. Stack trace:

0. 0xb2087bc Poco::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/asap/clickhouse/bin/clickhouse
1. 0x4d8e3c9 DB::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/asap/clickhouse/bin/clickhouse
2. 0x89022c0 DB::MergeTreeDataWriter::splitBlockIntoParts(DB::Block const&, unsigned long)  in /data/asap/clickhouse/bin/clickhouse
3. 0x8962287 DB::ReplicatedMergeTreeBlockOutputStream::write(DB::Block const&)  in /data/asap/clickhouse/bin/clickhouse
4. 0x8d83f4e DB::PushingToViewsBlockOutputStream::write(DB::Block const&)  in /data/asap/clickhouse/bin/clickhouse
5. 0x8d9a330 DB::SquashingBlockOutputStream::writeSuffix()  in /data/asap/clickhouse/bin/clickhouse
6. 0x4e09d02 DB::TCPHandler::processInsertQuery(DB::Settings const&)  in /data/asap/clickhouse/bin/clickhouse
7. 0x4e0b2bb DB::TCPHandler::runImpl()  in /data/asap/clickhouse/bin/clickhouse
8. 0x4e0b769 DB::TCPHandler::run()  in /data/asap/clickhouse/bin/clickhouse
9. 0x94f4177 Poco::Net::TCPServerConnection::start()  in /data/asap/clickhouse/bin/clickhouse
10. 0x94f455b Poco::Net::TCPServerDispatcher::run()  in /data/asap/clickhouse/bin/clickhouse
11. 0xb27cb3f Poco::PooledThread::run()  in /data/asap/clickhouse/bin/clickhouse
12. 0xb279557 Poco::ThreadImpl::runnableEntry(void*)  in /data/asap/clickhouse/bin/clickhouse
13. 0xb27b526 ?  in /data/asap/clickhouse/bin/clickhouse
14. 0x7dd5 start_thread  in /usr/lib64/libpthread-2.17.so
15. 0xfeb3d __clone  in /usr/lib64/libc-2.17.so
, Stack trace (when copying this message, always include the lines below):

0. 0xb2087bc Poco::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/asap/clickhouse/bin/clickhouse
1. 0x4d8e3c9 DB::Exception::Exception(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, int)  in /data/asap/clickhouse/bin/clickhouse
2. 0x4dae15a DB::readException(DB::ReadBuffer&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&)  in /data/asap/clickhouse/bin/clickhouse
3. 0x89cf3a7 DB::Connection::receiveException()  in /data/asap/clickhouse/bin/clickhouse
4. 0x89d38ed DB::Connection::receivePacket()  in /data/asap/clickhouse/bin/clickhouse
5. 0x8d89997 DB::RemoteBlockOutputStream::writeSuffix()  in /data/asap/clickhouse/bin/clickhouse
6. 0x8843108 DB::StorageDistributedDirectoryMonitor::processFile(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&)  in /data/asap/clickhouse/bin/clickhouse
7. 0x8846b85 DB::StorageDistributedDirectoryMonitor::processFiles()  in /data/asap/clickhouse/bin/clickhouse
8. 0x8846ee8 DB::StorageDistributedDirectoryMonitor::run()  in /data/asap/clickhouse/bin/clickhouse
9. 0x8848a2d ThreadFromGlobalPool::ThreadFromGlobalPool<void (DB::StorageDistributedDirectoryMonitor::*)(), DB::StorageDistributedDirectoryMonitor*>(void (DB::StorageDistributedDirectoryMonitor::*&&)(), DB::StorageDistributedDirectoryMonitor*&&)::'lambda'()::operator()() const  in /data/asap/clickhouse/bin/clickhouse
10. 0x4dc7eca ThreadPoolImpl<std::__1::thread>::worker(std::__1::__list_iterator<std::__1::thread, void*>)  in /data/asap/clickhouse/bin/clickhouse
11. 0x4dc69dc ?  in /data/asap/clickhouse/bin/clickhouse
12. 0x7dd5 start_thread  in /usr/lib64/libpthread-2.17.so
13. 0xfeb3d __clone  in /usr/lib64/libc-2.17.so
 (version 20.1.8.41)
``` 

Insert进行分布式执行时, 在其他节点报错, 涉及以下多个分布式节点: 

```
CK10:24902.
CK11:24902.
CK12:24902.
CK3:24902.
CK4:24902.
CK5:24902.
CK6:24902.
CK7:24902.
CK8:24902.
CK9:24902.
``` 

建议减小单个Insert语句涉及的分区数
