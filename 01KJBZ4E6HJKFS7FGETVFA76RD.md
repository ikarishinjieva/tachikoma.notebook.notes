---
title: 20231205 - 整理OMS的全量验证链路
confluence_page_id: 2589847
created_at: 2023-12-05T16:15:48+00:00
updated_at: 2023-12-13T06:23:05+00:00
---

目录

# 工作目录

/u01/ds/run/10.186.16.126-9000:90215:0000000001

# 进程

```
ds       25679  108  0.1 18881056 486012 pts/0 Sl   23:14   0:10 /opt/alibaba/java/bin/java -server -Xms8g -Xmx8g -Xmn4g -Xss512k -XX:ErrorFile=/u01/ds/bin/..//run//10.186.16.126-9000:90215:0000000001/logs/hs_err_pid%p.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/u01/ds/bin/..//run//10.186.16.126-9000:90215:0000000001/logs -verbose:gc -Xloggc:/u01/ds/bin/..//run//10.186.16.126-9000:90215:0000000001/logs/gc_%p.log -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Dlogback.configurationFile=/home/ds/plugins/verifier/conf_template/logback.xml -classpath /home/ds/plugins/verifier/verifier.jar:conf com.oceanbase.verify.Verify -t 10.186.16.126-9000:90215:0000000001 -c /home/ds/run/10.186.16.126-9000:90215:0000000001/conf/checker.conf start
``` 

# 配置文件

```
[root@R740-26 logs]# grep "" /home/ds/run/10.186.16.126-9000:90215:0000000001/conf/checker.conf
condition.whiteCondition=[{"name":"OBTEST","map":"OBTEST","all":false,"sub":[{"name":"CUSTOMER","map":"CUSTOMER"},{"name":"DISTRICT","map":"DISTRICT"},{"name":"HISTORY","map":"HISTORY"},{"name":"ITEM","map":"ITEM"},{"name":"NEW_ORDER","map":"NEW_ORDER"},{"name":"ORDERS","map":"ORDERS"},{"name":"ORDER_LINE","map":"ORDER_LINE"},{"name":"STOCK","map":"STOCK"},{"name":"WAREHOUSE","map":"WAREHOUSE"}]}]
datasource.master.type=ORACLE
datasource.master.address=10.186.16.126:1521/EE.ORACLE.DOCKER
datasource.master.username=OBTEST
datasource.master.password=111111
datasource.image.region=
datasource.image.type=OB_IN_ORACLE_MODE
datasource.image.address=10.186.16.126:32883
datasource.image.username=OBTEST@oms_oracle#obthree
datasource.image.password=111111
datasource.master.region=
datasource.timezone=+08:00
filter.master.blacklist=
filter.master.whitelist=*.*
force.split.by.rowid=true
limitator.reviewer.period=3
limitator.reviewer.review.batch.max=100
limitator.reviewer.time.max=60
limitator.select.batch.max=600
limitator.table.diff.max=10000
limitator.platform.threads.number=8
limitator.datasource.connections.max=50
limitator.datasource.image.ob10freememory.min=20
limitator.reviewer.rounds.max=20
limitator.resume.verify.fromkeys=false
mapper.from_master_to_image.list=
rectifier.image.enable=false
rectifier.image.operator.update=false
rectifier.image.operator.delete=false
rectifier.image.operator.insert=false
sampler.verify.ratio=100
task.split.mode=false
task.type=verify
task.checker_jvm_param=-server -Xms8g -Xmx8g -Xmn4g -Xss512k
task.id=90215
task.subId=2
task.resume=false
``` 

  

配置转换的逻辑 (文件 -> 内存, 不是一一对应): com.oceanbase.verify.common.config.ConfigConverter#buildSourceConfig

# 概念

  - SourceGroup: 包括 多个数据源Source (源端和目标端)
  - InfoExchanger/SliceExchanger: "汇率中心", 完成源端和目标端的差异处理, 比如表结构不同会导致查询条件的差异
  - CacheCollection: 用作 Split 和 Verify 过程中的数据存储
  - SourceExchangeInfo: 记录数据源的状态 (是否读取完成, 是否是源端, 记录检查点等) (不确定为什么要跟数据源分离)
  - ConditionSourceConnectTask: 针对某个数据源, 从数据源中获取数据的线程  

  

# 结构梳理

Split阶段: 

  - SourceGroup
    - Source(源端数据库)
      - TriggerSliceProvider (切片划分器, 主动对 源端数据 划分切片)
      - 线程: ConditionSourceConnectTask (获取切片条件, 将切片条件传递给目标端切片器, 并在源端查询切片数据)
        - 具体逻辑: 
          - 获取切片条件 (TriggerSliceProvider#next)  

          - 如果不使用 InMod, 将切片条件传递给目标端切片器 (sourceGroup.InfoExchanger.communicate)
          - 将 修订后的查询条件, 去 数据源中 查询 (source#poll)
          - 将查询结果进行包装 (DataflowSource#wrapRecordBatch), 输出到 DataflowSource.readyQueue
          - 获取数据后进行回调: 如果使用InMod, 将获取的数据中的主键信息拼接成查询条件, 传递给目标端的切片器 (ConditionSourceConnectTask#afterReadCallback -> SliceExchanger#communicateIndexValues)
      - 输出队列: DataflowSource.readyQueue  

      - 线程组: source_master_drive_worker_n, 执行ShuffleCallable任务 (ShuffleCallable#call)
        - batchProcessor 对数据进行修订  

        - shuffler 对数据进行排序
        - 进行流控 (AbstractShuffler#shuffle)
        - 将排序结果 写入 CacheCollection  (AbstractShuffler#shuffle)
  

    - Source(目标端数据库)
      - PassiveSliceProvider (切片划分器, 被动接受分片, 提供给目标数据库用作查询)  

        - 接受切片的代码: SliceExchanger#communicate 或 communicateIndexValues (通过 汇率中心 进行切片的传递)
      - 线程组: source_image_drive_worker_n, 任务ConditionSourceConnectTask
        - 具体逻辑: 
          - 获取切片条件 (PassiveSliceProvider#next)
          - 将 修订后的查询条件, 去 数据源中 查询 (source#poll)
          - 将查询结果进行包装 (DataflowSource#wrapRecordBatch), 输出到 DataflowSource.readyQueue  

      - 输出队列: DataflowSource.readyQueue
      - 线程组: ShuffleTask-n, 执行ShuffleCallable任务 (ShuffleCallable#call)
        - batchProcessor 对数据进行修订  

        - shuffler 对数据进行排序  

        - 进行流控 (AbstractShuffler#shuffle)
        - 将排序结果 写入 CacheCollection  (AbstractShuffler#shuffle)
    - 流控机制: 
      - BackPressureController线程
      - 周期性更新 流控策略 (如果 内存使用率 或 RPS 达到阈值, 则打标记"开始流控". 实际流控是在shuffle中进行)

Verify阶段: 

  - verifyWorker.run, 生成线程池 VerifyTask-n, 任务: VerifyThread
    - RecordVerify#prepare: 获取 源端的CacheCollection 和 目标端的CacheCollection, 作为数据源
    - RecordVerify#verify: 进行两个数据源的校验动作 (RecordVerify#verifyTwoSource)
      - 四种verify level: (配置项 verifyLevel, 默认值 CHECK_DML_SEQUENCE)
        - CHECK_EXIST (没作用)
        - CHECK_FINAL_RESULT (没作用)
        - CHECK_DML_SEQUENCE (比较DML操作, 用作增量流的比较?)
        - CHECK_SLICE (默认值, 对数据进行比较, 以下以此为例)
      - 逻辑: 
        - 从 源端 CacheCollection 获取 一个数据切片, 找到目标端 CacheCollection中与之对应的切片
        - 对两个切片进行比较
        - verify结束后, 从CacheCollection中去除已检查的数据 (CacheCollection的大小限制 依靠 流控机制)
        - verify结束后, 更新检查点 (将 排序检查点 更新为 数据源检查点, 下次从该检查点开始获取数据)

# 线程列表

```
[arthas@22320]$ thread -n 1000
"VerifyTask-5" Id=86 cpuUsage=99.84% deltaTime=201ms time=27307ms RUNNABLE
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyBatch(RecordVerify.java:365)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyTwoSource(RecordVerify.java:242)
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:145)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"VerifyTask-0" Id=76 cpuUsage=99.37% deltaTime=200ms time=27053ms RUNNABLE
    at com.oceanbase.verify.worker.verify.dataset.OmsRecordSliceCompareOp.compareAndProcess(OmsRecordSliceCompareOp.java:21)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyBatch(RecordVerify.java:341)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyTwoSource(RecordVerify.java:242)
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:145)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"VerifyTask-4" Id=84 cpuUsage=98.6% deltaTime=199ms time=26751ms RUNNABLE
    at java.util.concurrent.ConcurrentHashMap$Traverser.advance(ConcurrentHashMap.java:3339)
    at java.util.concurrent.ConcurrentHashMap$BaseIterator.<init>(ConcurrentHashMap.java:3391)
    at java.util.concurrent.ConcurrentHashMap$EntryIterator.<init>(ConcurrentHashMap.java:3450)
    at java.util.concurrent.ConcurrentHashMap$EntrySetView.iterator(ConcurrentHashMap.java:4746)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyBatch(RecordVerify.java:270)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyTwoSource(RecordVerify.java:242)
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:145)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"VerifyTask-6" Id=88 cpuUsage=83.07% deltaTime=168ms time=25612ms RUNNABLE
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:145)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"VerifyTask-2" Id=80 cpuUsage=52.81% deltaTime=106ms time=27105ms BLOCKED on com.oceanbase.oms.connector.source.dataflow.TableInfo@2f9b0dd5 owned by "VerifyTask-3" Id=82
    at com.oceanbase.oms.connector.source.dataflow.VerifyBatchListener.onNewBatchState(VerifyBatchListener.java:77)
    -  blocked on com.oceanbase.oms.connector.source.dataflow.TableInfo@2f9b0dd5
    at com.oceanbase.verify.connectors.common.batchlistener.VerifySourceStateListener.consumeDone(VerifySourceStateListener.java:58)
    at com.oceanbase.verify.connectors.common.batchlistener.VerifySourceStateListener.onNewBatchState(VerifySourceStateListener.java:33)
    at com.oceanbase.verify.core.utils.CoreUtil.invokeBatchCallBack(CoreUtil.java:90)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyBatch(RecordVerify.java:304)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyTwoSource(RecordVerify.java:242)
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:145)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"source_image_drive_worker_3" Id=72 cpuUsage=50.48% deltaTime=102ms time=9174ms TIMED_WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@25e07759
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@25e07759
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:216)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2087)
    at java.util.concurrent.ArrayBlockingQueue.poll(ArrayBlockingQueue.java:418)
    at com.oceanbase.verify.connectors.sourcegroup.provider.PassiveSliceProvider.next(PassiveSliceProvider.java:71)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:53)
    at java.lang.Thread.run(Thread.java:853)

"VerifyTask-1" Id=78 cpuUsage=32.15% deltaTime=65ms time=28121ms BLOCKED on com.oceanbase.oms.connector.source.dataflow.TableInfo@2f9b0dd5 owned by "VerifyTask-3" Id=82
    at com.oceanbase.oms.connector.source.dataflow.VerifyBatchListener.onNewBatchState(VerifyBatchListener.java:77)
    -  blocked on com.oceanbase.oms.connector.source.dataflow.TableInfo@2f9b0dd5
    at com.oceanbase.verify.connectors.common.batchlistener.VerifySourceStateListener.consumeDone(VerifySourceStateListener.java:58)
    at com.oceanbase.verify.connectors.common.batchlistener.VerifySourceStateListener.onNewBatchState(VerifySourceStateListener.java:33)
    at com.oceanbase.verify.core.utils.CoreUtil.invokeBatchCallBack(CoreUtil.java:90)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyBatch(RecordVerify.java:304)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyTwoSource(RecordVerify.java:242)
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:145)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"source_master_drive_worker_3" Id=68 cpuUsage=26.75% deltaTime=54ms time=12047ms RUNNABLE
    at sun.nio.ch.NativeThread.current(Native Method)
    at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:339)
    at oracle.net.nt.TimeoutSocketChannel.read(TimeoutSocketChannel.java:174)
    at oracle.net.ns.NSProtocolNIO.doSocketRead(NSProtocolNIO.java:555)
    at oracle.net.ns.NIOPacket.readHeader(NIOPacket.java:258)
    at oracle.net.ns.NIOPacket.readPacketFromSocketChannel(NIOPacket.java:190)
    at oracle.net.ns.NIOPacket.readFromSocketChannel(NIOPacket.java:132)
    at oracle.net.ns.NIOPacket.readFromSocketChannel(NIOPacket.java:105)
    at oracle.net.ns.NIONSDataChannel.readDataFromSocketChannel(NIONSDataChannel.java:91)
    at oracle.jdbc.driver.T4CMAREngineNIO.prepareForUnmarshall(T4CMAREngineNIO.java:784)
    at oracle.jdbc.driver.T4CMAREngineNIO.getNBytes(T4CMAREngineNIO.java:598)
    at oracle.jdbc.driver.T4CMAREngineNIO.unmarshalNBytes(T4CMAREngineNIO.java:567)
    at oracle.jdbc.driver.DynamicByteArray.unmarshalBuffer(DynamicByteArray.java:295)
    at oracle.jdbc.driver.DynamicByteArray.unmarshalCLR(DynamicByteArray.java:210)
    at oracle.jdbc.driver.T4CNumberAccessor.unmarshalBytes(T4CNumberAccessor.java:193)
    at oracle.jdbc.driver.T4CNumberAccessor.unmarshalOneRow(T4CNumberAccessor.java:175)
    at oracle.jdbc.driver.T4CTTIrxd.unmarshal(T4CTTIrxd.java:1594)
    at oracle.jdbc.driver.T4CTTIrxd.unmarshal(T4CTTIrxd.java:1313)
    at oracle.jdbc.driver.T4C8Oall.readRXD(T4C8Oall.java:888)
    at oracle.jdbc.driver.T4CTTIfun.receive(T4CTTIfun.java:468)
    at oracle.jdbc.driver.T4CTTIfun.doRPC(T4CTTIfun.java:269)
    at oracle.jdbc.driver.T4C8Oall.doOALL(T4C8Oall.java:655)
    at oracle.jdbc.driver.T4CPreparedStatement.doOall8(T4CPreparedStatement.java:270)
    at oracle.jdbc.driver.T4CPreparedStatement.fetch(T4CPreparedStatement.java:1079)
    at oracle.jdbc.driver.OracleStatement.fetchMoreRows(OracleStatement.java:3456)
    at oracle.jdbc.driver.InsensitiveScrollableResultSet.fetchMoreRows(InsensitiveScrollableResultSet.java:742)
    at oracle.jdbc.driver.InsensitiveScrollableResultSet.absoluteInternal(InsensitiveScrollableResultSet.java:698)
    at oracle.jdbc.driver.InsensitiveScrollableResultSet.next(InsensitiveScrollableResultSet.java:412)
    at com.alibaba.druid.pool.DruidPooledResultSet.next(DruidPooledResultSet.java:68)
    at com.oceanbase.oms.dataflow.common.stream.StreamRecordFetcher.hasNext(StreamRecordFetcher.java:66)
    at com.oceanbase.oms.dataflow.common.stream.StreamRecordBatchBuilder.hasNext(StreamRecordBatchBuilder.java:49)
    at com.oceanbase.oms.dataflow.common.stream.StreamRecordBatchBuilder.next(StreamRecordBatchBuilder.java:91)
    at com.oceanbase.oms.dataflow.common.stream.StreamRecordBatchBuilder.next(StreamRecordBatchBuilder.java:18)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:76)
    at java.lang.Thread.run(Thread.java:853)

"ShuffleTask-1" Id=92 cpuUsage=22.77% deltaTime=46ms time=15235ms WAITING on java.util.concurrent.locks.ReentrantLock$NonfairSync@1d911318 owned by "VerifyTask-3" Id=82
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.ReentrantLock$NonfairSync@1d911318
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:842)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:876)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1207)
    at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:209)
    at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)
    at ch.qos.logback.core.OutputStreamAppender.writeBytes(OutputStreamAppender.java:197)
    at ch.qos.logback.core.OutputStreamAppender.subAppend(OutputStreamAppender.java:231)
    at ch.qos.logback.core.rolling.RollingFileAppender.subAppend(RollingFileAppender.java:235)
    at ch.qos.logback.core.OutputStreamAppender.append(OutputStreamAppender.java:102)
    at ch.qos.logback.core.UnsynchronizedAppenderBase.doAppend(UnsynchronizedAppenderBase.java:84)
    at ch.qos.logback.core.spi.AppenderAttachableImpl.appendLoopOnAppenders(AppenderAttachableImpl.java:51)
    at ch.qos.logback.classic.Logger.appendLoopOnAppenders(Logger.java:270)
    at ch.qos.logback.classic.Logger.callAppenders(Logger.java:257)
    at ch.qos.logback.classic.Logger.buildLoggingEventAndAppend(Logger.java:421)
    at ch.qos.logback.classic.Logger.filterAndLog_2(Logger.java:414)
    at ch.qos.logback.classic.Logger.error(Logger.java:530)
    at com.oceanbase.verify.core.record.FullVerifyRecordBatch$RecordSizeDetector.detectRecordSize(FullVerifyRecordBatch.java:121)
    at com.oceanbase.verify.core.record.FullVerifyRecordBatch.add(FullVerifyRecordBatch.java:56)
    at com.oceanbase.verify.connectors.common.cache.MemoryFullRecordCache.put(MemoryFullRecordCache.java:39)
    at com.oceanbase.verify.connectors.common.cache.MemoryFullRecordCache.put(MemoryFullRecordCache.java:17)
    at com.oceanbase.verify.split.shuffle.AbstractShuffler.shuffle(AbstractShuffler.java:47)
    at com.oceanbase.verify.split.ShuffleCallable.call(ShuffleCallable.java:94)
    at com.oceanbase.verify.split.ShuffleCallable.call(ShuffleCallable.java:32)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"C2 CompilerThread9" [Internal] cpuUsage=16.76% deltaTime=33ms time=2180ms

"source_master_drive_worker_2" Id=67 cpuUsage=15.83% deltaTime=32ms time=11777ms WAITING on java.util.concurrent.locks.ReentrantLock$NonfairSync@1d911318 owned by "VerifyTask-3" Id=82
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.ReentrantLock$NonfairSync@1d911318
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:842)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:876)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1207)
    at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:209)
    at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)
    at ch.qos.logback.core.OutputStreamAppender.writeBytes(OutputStreamAppender.java:197)
    at ch.qos.logback.core.OutputStreamAppender.subAppend(OutputStreamAppender.java:231)
    at ch.qos.logback.core.rolling.RollingFileAppender.subAppend(RollingFileAppender.java:235)
    at ch.qos.logback.core.OutputStreamAppender.append(OutputStreamAppender.java:102)
    at ch.qos.logback.core.UnsynchronizedAppenderBase.doAppend(UnsynchronizedAppenderBase.java:84)
    at ch.qos.logback.core.spi.AppenderAttachableImpl.appendLoopOnAppenders(AppenderAttachableImpl.java:51)
    at ch.qos.logback.classic.Logger.appendLoopOnAppenders(Logger.java:270)
    at ch.qos.logback.classic.Logger.callAppenders(Logger.java:257)
    at ch.qos.logback.classic.Logger.buildLoggingEventAndAppend(Logger.java:421)
    at ch.qos.logback.classic.Logger.filterAndLog_1(Logger.java:398)
    at ch.qos.logback.classic.Logger.info(Logger.java:583)
    at com.oceanbase.oms.connector.source.dataflow.SinkCheckpointStore.addCheckpoint(SinkCheckpointStore.java:61)
    at com.oceanbase.verify.connectors.sourcegroup.exchange.SliceExchanger.communicate(SliceExchanger.java:157)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.DataflowSource.communicateSliceInfo(DataflowSource.java:159)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.DataflowSource$$Lambda$308/1016633682.communicatePartitionInfo(Unknown Source)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:59)
    at java.lang.Thread.run(Thread.java:853)

"source_master_drive_worker_1" Id=66 cpuUsage=6.79% deltaTime=13ms time=10497ms BLOCKED on com.oceanbase.oms.connector.source.dataflow.SinkCheckpointStore@29aaefb4 owned by "source_master_drive_worker_2" Id=67
    at com.oceanbase.oms.connector.source.dataflow.SinkCheckpointStore.addCheckpoint(SinkCheckpointStore.java:31)
    -  blocked on com.oceanbase.oms.connector.source.dataflow.SinkCheckpointStore@29aaefb4
    at com.oceanbase.verify.connectors.sourcegroup.exchange.SliceExchanger.communicate(SliceExchanger.java:157)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.DataflowSource.communicateSliceInfo(DataflowSource.java:159)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.DataflowSource$$Lambda$308/1016633682.communicatePartitionInfo(Unknown Source)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:59)
    at java.lang.Thread.run(Thread.java:853)

"source_master_drive_worker_4" Id=69 cpuUsage=2.94% deltaTime=5ms time=11053ms BLOCKED on com.oceanbase.oms.connector.source.dataflow.SinkCheckpointStore@29aaefb4 owned by "source_master_drive_worker_2" Id=67
    at com.oceanbase.oms.connector.source.dataflow.SinkCheckpointStore.addCheckpoint(SinkCheckpointStore.java:31)
    -  blocked on com.oceanbase.oms.connector.source.dataflow.SinkCheckpointStore@29aaefb4
    at com.oceanbase.verify.connectors.sourcegroup.exchange.SliceExchanger.communicate(SliceExchanger.java:157)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.DataflowSource.communicateSliceInfo(DataflowSource.java:159)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.DataflowSource$$Lambda$308/1016633682.communicatePartitionInfo(Unknown Source)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:59)
    at java.lang.Thread.run(Thread.java:853)

"arthas-command-execute" Id=98 cpuUsage=1.04% deltaTime=2ms time=91ms RUNNABLE
    at sun.management.ThreadImpl.dumpThreads0(Native Method)
    at sun.management.ThreadImpl.getThreadInfo(ThreadImpl.java:473)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.processTopBusyThreads(ThreadCommand.java:206)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.process(ThreadCommand.java:122)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.process(AnnotatedCommandImpl.java:82)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.access$100(AnnotatedCommandImpl.java:18)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:111)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:108)
    at com.taobao.arthas.core.shell.system.impl.ProcessImpl$CommandProcessTask.run(ProcessImpl.java:385)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"Thread-9" Id=53 cpuUsage=0.64% deltaTime=1ms time=283ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.verify.core.metrics.BackPressureController.run(BackPressureController.java:113)

"C1 CompilerThread12" [Internal] cpuUsage=0.51% deltaTime=1ms time=958ms

"source_image_drive_worker_2" Id=71 cpuUsage=0.46% deltaTime=0ms time=9199ms TIMED_WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@25e07759
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@25e07759
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:216)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2087)
    at java.util.concurrent.ArrayBlockingQueue.poll(ArrayBlockingQueue.java:418)
    at com.oceanbase.verify.connectors.sourcegroup.provider.PassiveSliceProvider.next(PassiveSliceProvider.java:71)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:53)
    at java.lang.Thread.run(Thread.java:853)

"source_image_drive_worker_1" Id=70 cpuUsage=0.46% deltaTime=0ms time=9127ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.oms.connector.common.util.TimeUtil.sleepMs(TimeUtil.java:45)
    at com.oceanbase.oms.connector.common.model.IPoll.tryWaitFill(IPoll.java:11)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:55)
    at java.lang.Thread.run(Thread.java:853)

"source_image_drive_worker_4" Id=73 cpuUsage=0.45% deltaTime=0ms time=9953ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.oms.connector.common.util.TimeUtil.sleepMs(TimeUtil.java:45)
    at com.oceanbase.oms.connector.common.model.IPoll.tryWaitFill(IPoll.java:11)
    at com.oceanbase.verify.connectors.sourcegroup.source.connectorsource.taskmanager.ConditionSourceConnectTask.run(ConditionSourceConnectTask.java:55)
    at java.lang.Thread.run(Thread.java:853)

"C1 CompilerThread17" [Internal] cpuUsage=0.22% deltaTime=0ms time=846ms

"C1 CompilerThread14" [Internal] cpuUsage=0.19% deltaTime=0ms time=906ms

"C1 CompilerThread13" [Internal] cpuUsage=0.13% deltaTime=0ms time=887ms

"C2 CompilerThread7" [Internal] cpuUsage=0.06% deltaTime=0ms time=3845ms

"C2 CompilerThread11" [Internal] cpuUsage=0.06% deltaTime=0ms time=1618ms

"C2 CompilerThread1" [Internal] cpuUsage=0.05% deltaTime=0ms time=2461ms

"VM Periodic Task Thread" [Internal] cpuUsage=0.05% deltaTime=0ms time=52ms

"C2 CompilerThread0" [Internal] cpuUsage=0.05% deltaTime=0ms time=2362ms

"Thread-8" Id=52 cpuUsage=0.05% deltaTime=0ms time=45ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.verify.split.Split.run(Split.java:183)

"C2 CompilerThread4" [Internal] cpuUsage=0.05% deltaTime=0ms time=1902ms

"C2 CompilerThread6" [Internal] cpuUsage=0.04% deltaTime=0ms time=2137ms

"C2 CompilerThread2" [Internal] cpuUsage=0.04% deltaTime=0ms time=2262ms

"C2 CompilerThread3" [Internal] cpuUsage=0.04% deltaTime=0ms time=2407ms

"C2 CompilerThread10" [Internal] cpuUsage=0.04% deltaTime=0ms time=1549ms

"C2 CompilerThread8" [Internal] cpuUsage=0.03% deltaTime=0ms time=3191ms

"C1 CompilerThread15" [Internal] cpuUsage=0.03% deltaTime=0ms time=882ms

"C1 CompilerThread16" [Internal] cpuUsage=0.03% deltaTime=0ms time=946ms

"C2 CompilerThread5" [Internal] cpuUsage=0.03% deltaTime=0ms time=1216ms

"VerifyWorker" Id=54 cpuUsage=0.03% deltaTime=0ms time=23ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.verify.worker.VerifyWorker.run(VerifyWorker.java:341)

"VerifyTask-7" Id=90 cpuUsage=0.02% deltaTime=0ms time=28016ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:139)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"Reference Handler" Id=2 cpuUsage=0.0% deltaTime=0ms time=12ms WAITING on java.lang.ref.Reference$Lock@5e28aaf1
    at java.lang.Object.wait(Native Method)
    -  waiting on java.lang.ref.Reference$Lock@5e28aaf1
    at java.lang.Object.wait(Object.java:502)
    at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
    at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"Finalizer" Id=3 cpuUsage=0.0% deltaTime=0ms time=14ms WAITING on java.lang.ref.ReferenceQueue$Lock@785454d2
    at java.lang.Object.wait(Native Method)
    -  waiting on java.lang.ref.ReferenceQueue$Lock@785454d2
    at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
    at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
    at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:287)

"Signal Dispatcher" Id=5 cpuUsage=0.0% deltaTime=0ms time=0ms RUNNABLE

"Attach Listener" Id=39 cpuUsage=0.0% deltaTime=0ms time=18ms RUNNABLE

"arthas-timer" Id=41 cpuUsage=0.0% deltaTime=0ms time=0ms WAITING on java.util.TaskQueue@6d489113
    at java.lang.Object.wait(Native Method)
    -  waiting on java.util.TaskQueue@6d489113
    at java.lang.Object.wait(Object.java:502)
    at java.util.TimerThread.mainLoop(Timer.java:526)
    at java.util.TimerThread.run(Timer.java:505)

"arthas-NettyHttpTelnetBootstrap-3-1" Id=44 cpuUsage=0.0% deltaTime=0ms time=46ms RUNNABLE (in native)
    at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
    at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:280)
    at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:96)
    at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:101)
    at com.alibaba.arthas.deps.io.netty.channel.nio.SelectedSelectionKeySetSelector.select(SelectedSelectionKeySetSelector.java:68)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.select(NioEventLoop.java:879)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:526)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997)
    at com.alibaba.arthas.deps.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    at java.lang.Thread.run(Thread.java:853)

"arthas-NettyWebsocketTtyBootstrap-4-1" Id=45 cpuUsage=0.0% deltaTime=0ms time=3ms RUNNABLE (in native)
    at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
    at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:280)
    at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:96)
    at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:101)
    at com.alibaba.arthas.deps.io.netty.channel.nio.SelectedSelectionKeySetSelector.select(SelectedSelectionKeySetSelector.java:68)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.select(NioEventLoop.java:879)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:526)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997)
    at com.alibaba.arthas.deps.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    at java.lang.Thread.run(Thread.java:853)

"arthas-NettyWebsocketTtyBootstrap-4-2" Id=46 cpuUsage=0.0% deltaTime=0ms time=3ms RUNNABLE (in native)
    at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
    at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:280)
    at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:96)
    at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:101)
    at com.alibaba.arthas.deps.io.netty.channel.nio.SelectedSelectionKeySetSelector.select(SelectedSelectionKeySetSelector.java:68)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.select(NioEventLoop.java:879)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:526)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997)
    at com.alibaba.arthas.deps.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    at java.lang.Thread.run(Thread.java:853)

"arthas-shell-server" Id=47 cpuUsage=0.0% deltaTime=0ms time=1ms TIMED_WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@13611441
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@13611441
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:216)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2087)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"arthas-session-manager" Id=48 cpuUsage=0.0% deltaTime=0ms time=0ms TIMED_WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@8854bfe
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@8854bfe
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:216)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2087)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"arthas-UserStat" Id=49 cpuUsage=0.0% deltaTime=0ms time=0ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@522d6cf8
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@522d6cf8
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:446)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"arthas-NettyHttpTelnetBootstrap-3-2" Id=51 cpuUsage=0.0% deltaTime=0ms time=207ms RUNNABLE (in native)
    at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
    at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:280)
    at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:96)
    at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
    at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:101)
    at com.alibaba.arthas.deps.io.netty.channel.nio.SelectedSelectionKeySetSelector.select(SelectedSelectionKeySetSelector.java:68)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.select(NioEventLoop.java:879)
    at com.alibaba.arthas.deps.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:526)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997)
    at com.alibaba.arthas.deps.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
    at com.alibaba.arthas.deps.io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    at java.lang.Thread.run(Thread.java:853)

"main" Id=1 cpuUsage=0.0% deltaTime=0ms time=2972ms WAITING on com.oceanbase.verify.worker.VerifyWorker@7b44bfa7
    at java.lang.Object.wait(Native Method)
    -  waiting on com.oceanbase.verify.worker.VerifyWorker@7b44bfa7
    at java.lang.Thread.join(Thread.java:1394)
    at java.lang.Thread.join(Thread.java:1468)
    at com.oceanbase.verify.Launch.join(Launch.java:221)
    at com.oceanbase.verify.Verify.main(Verify.java:40)

"AsyncAppender-Worker-ASYNC-SOURCE-MSG" Id=25 cpuUsage=0.0% deltaTime=0ms time=5ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@609c6f92
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@609c6f92
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at java.util.concurrent.ArrayBlockingQueue.take(ArrayBlockingQueue.java:403)
    at ch.qos.logback.core.AsyncAppenderBase$Worker.run(AsyncAppenderBase.java:289)

"AsyncAppender-Worker-ASYNC-SINK-MSG" Id=26 cpuUsage=0.0% deltaTime=0ms time=0ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@42e402c4
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@42e402c4
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at java.util.concurrent.ArrayBlockingQueue.take(ArrayBlockingQueue.java:403)
    at ch.qos.logback.core.AsyncAppenderBase$Worker.run(AsyncAppenderBase.java:289)

"logback-1" Id=27 cpuUsage=0.0% deltaTime=0ms time=125ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1081)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"logback-2" Id=28 cpuUsage=0.0% deltaTime=0ms time=6ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1081)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"Timer-0" Id=33 cpuUsage=0.0% deltaTime=0ms time=0ms WAITING on java.util.TaskQueue@25f0cd8e
    at java.lang.Object.wait(Native Method)
    -  waiting on java.util.TaskQueue@25f0cd8e
    at java.lang.Object.wait(Object.java:502)
    at java.util.TimerThread.mainLoop(Timer.java:526)
    at java.util.TimerThread.run(Timer.java:505)

"oracle.jdbc.driver.BlockSource.ThreadedCachingBlockSource.BlockReleaser" Id=34 cpuUsage=0.0% deltaTime=0ms time=0ms TIMED_WAITING on oracle.jdbc.driver.BlockSource$ThreadedCachingBlockSource$BlockReleaser@2693d65d
    at java.lang.Object.wait(Native Method)
    -  waiting on oracle.jdbc.driver.BlockSource$ThreadedCachingBlockSource$BlockReleaser@2693d65d
    at oracle.jdbc.driver.BlockSource$ThreadedCachingBlockSource$BlockReleaser.run(BlockSource.java:331)

"InterruptTimer" Id=35 cpuUsage=0.0% deltaTime=0ms time=4ms TIMED_WAITING on java.util.TaskQueue@44831269
    at java.lang.Object.wait(Native Method)
    -  waiting on java.util.TaskQueue@44831269
    at java.util.TimerThread.mainLoop(Timer.java:552)
    at java.util.TimerThread.run(Timer.java:505)

"OracleTimeoutPollingThread" Id=36 cpuUsage=0.0% deltaTime=0ms time=3ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at oracle.jdbc.driver.OracleTimeoutPollingThread.run(OracleTimeoutPollingThread.java:152)

"timerTaskScheduler" Id=56 cpuUsage=0.0% deltaTime=0ms time=145ms TIMED_WAITING on java.util.TaskQueue@b677a9
    at java.lang.Object.wait(Native Method)
    -  waiting on java.util.TaskQueue@b677a9
    at java.util.TimerThread.mainLoop(Timer.java:552)
    at java.util.TimerThread.run(Timer.java:505)

"DataFlowCheckpointThread" Id=65 cpuUsage=0.0% deltaTime=0ms time=81ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.oms.connector.common.util.TimeUtil.sleepMs(TimeUtil.java:45)
    at com.oceanbase.oms.connector.source.dataflow.DataFlowCheckpointManagerThread.run(DataFlowCheckpointManagerThread.java:25)
    at java.lang.Thread.run(Thread.java:853)

"Thread-30" Id=74 cpuUsage=0.0% deltaTime=0ms time=90ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.oceanbase.verify.split.ShuffleCheckpointThread.run(ShuffleCheckpointThread.java:45)

"VerifyTask-3" Id=82 cpuUsage=0.0% deltaTime=0ms time=27135ms RUNNABLE (in native)
    at java.io.FileOutputStream.writeBytes(Native Method)
    at java.io.FileOutputStream.write(FileOutputStream.java:326)
    at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
    at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
    at ch.qos.logback.core.recovery.ResilientOutputStreamBase.flush(ResilientOutputStreamBase.java:79)
    at ch.qos.logback.core.OutputStreamAppender.writeBytes(OutputStreamAppender.java:201)
    at ch.qos.logback.core.OutputStreamAppender.subAppend(OutputStreamAppender.java:231)
    at ch.qos.logback.core.rolling.RollingFileAppender.subAppend(RollingFileAppender.java:235)
    at ch.qos.logback.core.OutputStreamAppender.append(OutputStreamAppender.java:102)
    at ch.qos.logback.core.UnsynchronizedAppenderBase.doAppend(UnsynchronizedAppenderBase.java:84)
    at ch.qos.logback.core.spi.AppenderAttachableImpl.appendLoopOnAppenders(AppenderAttachableImpl.java:51)
    at ch.qos.logback.classic.Logger.appendLoopOnAppenders(Logger.java:270)
    at ch.qos.logback.classic.Logger.callAppenders(Logger.java:257)
    at ch.qos.logback.classic.Logger.buildLoggingEventAndAppend(Logger.java:421)
    at ch.qos.logback.classic.Logger.filterAndLog_0_Or3Plus(Logger.java:383)
    at ch.qos.logback.classic.Logger.info(Logger.java:591)
    at com.oceanbase.oms.connector.source.dataflow.VerifyBatchListener.onNewBatchState(VerifyBatchListener.java:87)
    at com.oceanbase.verify.connectors.common.batchlistener.VerifySourceStateListener.consumeDone(VerifySourceStateListener.java:58)
    at com.oceanbase.verify.connectors.common.batchlistener.VerifySourceStateListener.onNewBatchState(VerifySourceStateListener.java:33)
    at com.oceanbase.verify.core.utils.CoreUtil.invokeBatchCallBack(CoreUtil.java:90)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyBatch(RecordVerify.java:304)
    at com.oceanbase.verify.worker.verify.RecordVerify.verifyTwoSource(RecordVerify.java:242)
    at com.oceanbase.verify.worker.verify.RecordVerify.verify(RecordVerify.java:145)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:31)
    at com.oceanbase.verify.worker.VerifyThread.call(VerifyThread.java:13)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"ShuffleTask-0" Id=91 cpuUsage=0.0% deltaTime=0ms time=14014ms WAITING on java.util.concurrent.locks.ReentrantLock$NonfairSync@1d911318 owned by "VerifyTask-3" Id=82
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.ReentrantLock$NonfairSync@1d911318
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:842)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:876)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1207)
    at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:209)
    at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)
    at ch.qos.logback.core.OutputStreamAppender.writeBytes(OutputStreamAppender.java:197)
    at ch.qos.logback.core.OutputStreamAppender.subAppend(OutputStreamAppender.java:231)
    at ch.qos.logback.core.rolling.RollingFileAppender.subAppend(RollingFileAppender.java:235)
    at ch.qos.logback.core.OutputStreamAppender.append(OutputStreamAppender.java:102)
    at ch.qos.logback.core.UnsynchronizedAppenderBase.doAppend(UnsynchronizedAppenderBase.java:84)
    at ch.qos.logback.core.spi.AppenderAttachableImpl.appendLoopOnAppenders(AppenderAttachableImpl.java:51)
    at ch.qos.logback.classic.Logger.appendLoopOnAppenders(Logger.java:270)
    at ch.qos.logback.classic.Logger.callAppenders(Logger.java:257)
    at ch.qos.logback.classic.Logger.buildLoggingEventAndAppend(Logger.java:421)
    at ch.qos.logback.classic.Logger.filterAndLog_2(Logger.java:414)
    at ch.qos.logback.classic.Logger.error(Logger.java:530)
    at com.oceanbase.verify.core.record.FullVerifyRecordBatch$RecordSizeDetector.detectRecordSize(FullVerifyRecordBatch.java:121)
    at com.oceanbase.verify.core.record.FullVerifyRecordBatch.add(FullVerifyRecordBatch.java:56)
    at com.oceanbase.verify.connectors.common.cache.MemoryFullRecordCache.put(MemoryFullRecordCache.java:39)
    at com.oceanbase.verify.connectors.common.cache.MemoryFullRecordCache.put(MemoryFullRecordCache.java:17)
    at com.oceanbase.verify.split.shuffle.AbstractShuffler.shuffle(AbstractShuffler.java:47)
    at com.oceanbase.verify.split.ShuffleCallable.call(ShuffleCallable.java:94)
    at com.oceanbase.verify.split.ShuffleCallable.call(ShuffleCallable.java:32)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"Druid-ConnectionPool-Create-1328875296" Id=93 cpuUsage=0.0% deltaTime=0ms time=125ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@2389468c
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@2389468c
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2813)

"Druid-ConnectionPool-Destroy-1328875296" Id=94 cpuUsage=0.0% deltaTime=0ms time=1ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.alibaba.druid.pool.DruidDataSource$DestroyConnectionThread.run(DruidDataSource.java:2913)

"Druid-ConnectionPool-Create-1358902924" Id=95 cpuUsage=0.0% deltaTime=0ms time=13ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@1b905d9e
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@1b905d9e
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2813)

"Druid-ConnectionPool-Destroy-1358902924" Id=96 cpuUsage=0.0% deltaTime=0ms time=0ms TIMED_WAITING
    at java.lang.Thread.sleep0(Native Method)
    at java.lang.Thread.sleep(Thread.java:390)
    at com.alibaba.druid.pool.DruidDataSource$DestroyConnectionThread.run(DruidDataSource.java:2913)

"MariaDb-timeout-1" Id=97 cpuUsage=0.0% deltaTime=0ms time=14ms TIMED_WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@47dc14df
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@47dc14df
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:216)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2087)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"logback-3" Id=99 cpuUsage=0.0% deltaTime=0ms time=0ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1081)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"logback-4" Id=100 cpuUsage=0.0% deltaTime=0ms time=0ms WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park0(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@7b17fccc
    at sun.misc.Unsafe.park(Unsafe.java:1038)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:176)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2047)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1081)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:853)

"Gang worker#13 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#5 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#15 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2343ms

"Gang worker#31 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2319ms

"Gang worker#38 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2254ms

"Gang worker#43 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2385ms

"Service Thread" [Internal] cpuUsage=0.0% deltaTime=0ms time=6ms

"Gang worker#9 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#2 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=1ms

"Gang worker#5 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2217ms

"Gang worker#26 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2376ms

"Gang worker#30 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2265ms

"Gang worker#14 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2244ms

"Gang worker#27 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2207ms

"Gang worker#4 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2357ms

"Gang worker#42 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2089ms

"Gang worker#39 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2202ms

"Concurrent Mark-Sweep GC Thread" [Internal] cpuUsage=0.0% deltaTime=0ms time=872ms

"Gang worker#49 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2225ms

"Gang worker#2 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2252ms

"Gang worker#16 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2259ms

"Surrogate Locker Thread (Concurrent GC)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#25 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2466ms

"Gang worker#12 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#41 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2351ms

"Gang worker#4 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=1ms

"Gang worker#33 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2234ms

"Gang worker#3 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2347ms

"Gang worker#8 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#48 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2493ms

"Gang worker#1 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2380ms

"Gang worker#3 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#40 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2327ms

"Gang worker#17 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2314ms

"Gang worker#24 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2284ms

"Gang worker#32 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2284ms

"Gang worker#51 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2676ms

"Gang worker#47 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2323ms

"Gang worker#35 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2272ms

"Gang worker#0 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2462ms

"Gang worker#10 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2300ms

"Gang worker#18 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2290ms

"Gang worker#23 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2259ms

"Gang worker#8 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2206ms

"Gang worker#0 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=1ms

"Gang worker#7 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#46 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2199ms

"Gang worker#50 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2183ms

"Gang worker#11 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#22 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2239ms

"Gang worker#34 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2393ms

"Gang worker#9 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2208ms

"Gang worker#19 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2296ms

"Gang worker#37 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2436ms

"Gang worker#20 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2368ms

"Gang worker#45 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2396ms

"Gang worker#6 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2355ms

"Gang worker#21 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2296ms

"Gang worker#28 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2260ms

"Gang worker#6 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#13 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2262ms

"Gang worker#10 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=0ms

"Gang worker#44 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2235ms

"Gang worker#1 (Parallel CMS Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=1ms

"Gang worker#11 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2336ms

"Gang worker#36 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2185ms

"Gang worker#7 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2236ms

"VM Thread" [Internal] cpuUsage=0.0% deltaTime=0ms time=1065ms

"Gang worker#12 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2340ms

"Gang worker#29 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2356ms

"Gang worker#52 (Parallel GC Threads)" [Internal] cpuUsage=0.0% deltaTime=0ms time=2389ms

[arthas@22320]$ session (4117c94d-f354-463e-b91a-4413aaa7c305) is closed because server is going to shutdown.
bash-4.2$
``` 

  - VerifyTask-n, 上一节 已经解释
  - ShuffleTask-n, 上一节 已经解释
  - slice-worker-n, 与全量迁移类似, 切片器的线程组
  - source_image_drive_worker_n, 上一节 已经解释
  - source_master_drive_worker_1, 上一节 已经解释
  - DataFlowCheckpointThread, 在检查点机制一节描述 (DataFlowSliceProvider管理该线程)
  - Thread-n, 实际是 ShuffleCheckpointThread (FIXME: 线程名)
  - Thread-n, 实际是 Split (FIXME: 线程名)
  - Thread-n, 实际是 BackPressureController (FIXME: 线程名)  

# 日志分析

# 检查点机制

TODO: DataFlowSliceProvider

# 原始记录

```
com.oceanbase.verify.Verify#main
	-> com.oceanbase.verify.Launch#launch			
		-> 初始化 SourceGroup (com.oceanbase.verify.Launch#initSourceGroup)
			-> 初始化 sourceGroup.preprocessor. (类型是: FullPreprocessor)
				-> 比较源库和目标库的表结构是否一致, 判断是否使用IN Mode (比如: 如果表分区不完全匹配, 可以使用IN Mode来进行检查) (FullPreprocessor#compareTablePartition)
			-> 初始化 sourceGroup.InfoExchanger ("汇率中心", 完成源端和目标端的差异处理, 比如表结构不同会导致查询条件的差异)
		-> 初始化cache (cacheCollectionList), 用作之后Split和Verify过程中的数据存储 (Launch#initCacheCollection)
		-> 初始化sourceExchangeInfoCache, 记录数据源的状态 (是否读取完成, 是否是源端, 记录检查点等) (不确定为什么要跟数据源分离)
		-> 启动 ShuffleCheckpointThread 线程, 将数据源的状态 从 数据源 更新到 sourceExchangeInfoCache 中 (不确定其中作用)
		-> 启动 Split 线程
			-> 启动 sourceGroup (com.oceanbase.verify.split.Split#run)
				-> sourceGroup.InfoExchanger.start (类型是: SliceExchanger)
					-> TriggerSliceProvider.start (源端 数据源)
						-> DataFlowSliceProvider.start (与全量复制相同, 用作生成切片的边界)
					-> PassiveSliceProvider.start (目标端 数据源)
				-> 启动 source 线程 (类型是: DataflowSource)
					-> 启动 ConditionSourceTaskManager 管理器
						-> 启动 线程组, 任务: ConditionSourceConnectTask
							-> 线程逻辑: (ConditionSourceConnectTask#run)
								- 从 infoProvider.next() 获取查询条件 (实际是 sourceGroup.InfoExchanger.distribute().next() )
								- 使用 communicate.communicatePartitionInfo 修订信息 (实际是 sourceGroup.InfoExchanger.communicate, 根据源端的查询条件, 映射到目标端的查询条件)
								- 将 修订后的查询条件, 去 数据源中 查询
								- 将查询结果进行包装 (使用的是: DataflowSource#wrapRecordBatch), 输出到 DataflowSource.readyQueue
			-> 启动BackPressureController线程, 周期性更新 流控策略 (如果 内存使用率 或 RPS 达到阈值, 则打标记"开始流控". 实际流控是在shuffle中进行)
			-> 启动shuffle线程组, 执行ShuffleCallable任务
				-> 从 DataflowSource.readyQueue 获取切片
				-> 对切片数据进行排序
				-> 进行流控
				-> 将完成排序的切片, 送入cache
		-> this.verifyWorker.start()
			-> verifyWorker.run, 生成线程池, 任务: VerifyThread
				-> RecordVerify#prepare
					-> 获取 源端的cache 和 目标端的cache, 作为数据源
					-> FIXME: 此处代码非常别扭, 将cache和元数据分开获取
				-> RecordVerify#verify
					-> RecordVerify#verifyTwoSource
						-> RecordVerify#verifyBatch (比较源端和目标端的数据)
							- 从 源端 和 目标端 找到 两个数据组 (从cache中查找), 确认其是shuffle done的 (非shuffle done的数据不会再次进行扫描)
							- CompareOperator#compareAndProcess 比较两组数据是否一致
							- FIXME: 这里有一段特殊逻辑, 当源端数据扫描完成时, 对目标端的数据 确认在源端不存在, 并与null进行比较, 输出最终结果
```
