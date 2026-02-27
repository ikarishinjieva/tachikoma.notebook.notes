---
title: 20231108 - OMS研究, 阻塞目标库, 对队列进行探索
confluence_page_id: 2589388
created_at: 2023-11-08T05:28:39+00:00
updated_at: 2024-02-05T05:50:56+00:00
---

# 阻塞方式

压力脚本: [oracle.sql](/assets/01KJBZ3N9ACNJ631N9B3GRQK2H/oracle.sql)

在目标端的ob. oracle阻塞方式: 

```
start transaction;
LOCK TABLE OBTEST.T1 IN EXCLUSIVE MODE;
``` 

# 观测

使用 [20231025 - 通过arthas获取OMS队列监控] 的arthas脚本, 对队列进行观测, 结果: 

```
bash-4.2$ bash get_status.sh
    @String[fetcherOutQueueSize=0],
    @String[extractorOutQueueSize=0],
    @String[logAggregatorOutQueueSize=0],
    @String[preparedRecordOutQueueSize=0],
    @String[logRecordConverterOutQueueSize=0],
    @String[dispatchOutIdsSize=0],
    @String[dispatchOutRecordsSize=0],
    @String[dispatchOutQueueSize=0],
    @String[scnIncrementRecordQueueInSize=0],
    @String[sourceRecordQueueSize=0],
    @String[sourceOutQueueSize=32],
    @String[bridgeThread0OutQueueSize=32],
    @String[bridgeThread1OutQueueSize=16320],
    @String[sinkTasksCount=64],
``` 

发现在源端的队列都为0, 在目标端的队列在正确阻塞

### 目标端的数据来源

在目标端, 观测其数据来源, 通过arthas找到连接: 

```
[arthas@57517]$ vmtool --action getInstances --className alibaba.drcnet.impl.DRCNETClientImpl --express 'instances[0].connection.channel.strVal'
@String[[id: 0x7ab049b5, L:/10.186.16.126:44582 - R:/10.186.16.126:17002]]
``` 

  
在操作系统中找到进程: 

```
[root@R740-26 ~]# ss -anp | grep 44582
tcp    ESTAB      0      0      10.186.16.126:17002              10.186.16.126:44582               users:(("store",pid=74095,fd=23))
tcp    ESTAB      0      0        [::ffff:10.186.16.126]:44582                [::ffff:10.186.16.126]:17002               users:(("java",pid=57517,fd=587))
``` 

目标端是从store中获取数据, store的端口为17002

另外发现问题: 

```
[root@R740-26 ~]# ss -anlp | grep 17002
tcp    LISTEN     0      1024      *:17002                 *:*                   users:(("java",pid=74254,fd=9),("store",pid=74095,fd=9))
``` 

由于目标端是store的子进程, 在生成子进程的时候, 并没有良好地处理从父进程继承的端口

### 源端的目标去向

store的另一根连接: 

```
[root@R740-26 ~]# ss -anp | grep 44833
tcp    LISTEN     0      1024      *:44833                 *:*                   users:(("java",pid=74254,fd=21),("store",pid=74095,fd=21))
tcp    ESTAB      0      0      127.0.0.1:44833              127.0.0.1:40924               users:(("store",pid=74095,fd=22))
tcp    ESTAB      0      0        [::ffff:127.0.0.1]:40924                [::ffff:127.0.0.1]:44833               users:(("java",pid=74254,fd=606))
``` 

store监听端口: 44833

源端java 连接到 store

找到向store的操作对象: 

```
[arthas@74254]$ vmtool --action getInstances --className com.taobao.drc.IDRCDeliver --express "instances[0]
> "
@SimpleDRCDeliver[
    logger=@Logger[Logger[com.taobao.drc.deliver.SimpleDRCDeliver]],
    drcCMUrl=@String[],
    userInstanceName=@String[],
    storeAddress=@String[127.0.0.1],
    storePort=@Integer[44833],
    messageTransfer=@NettyTransfer[com.taobao.drc.transfer.netty.NettyTransfer@493bc115],
    transferToken=@DRCTransferToken[com.taobao.drc.transfer.DRCTransferToken@32d3aef0],
    drcMessageHub=@SimpleDRCMessageHub[com.taobao.drc.deliver.SimpleDRCMessageHub@8e00e22],
    isDirectConnectMode=@Boolean[true],
    connectorProps=@HashMap[isEmpty=false;size=18],
]
``` 

kafka connect的初始化代码: 

```
Worker worker = new DRCDeliverWorkerSourceTask(workerId, time, connectorFactory, config, new MemoryOffsetBackingStore(), sourceRecordConverterFactory, drcDeliver, breakPointInfo);
DRCDeliverHerder herder = new DRCDeliverHerder(worker);
Connect connect = new Connect(herder, rest);
 
try {
    connect.start();
    startConnector(herder, connectorProps);
} catch (Throwable var23) {
    log.error("Stopping after connector error", var23);
    connect.stop();
    stopLoop = true;
}
``` 

kafka connect的核心逻辑: 

![image2023-11-8 15:39:4.png](/assets/01KJBZ3N9ACNJ631N9B3GRQK2H/image2023-11-8%2015%3A39%3A4.png)

从上一队列获取数据: 

```
ts=2023-11-08 15:35:22;thread_name=pool-2-thread-1;id=2f;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@764c12b6
    @com.taobao.drc.logminer.connect.LogMinerConnectorTask.poll()
        at org.apache.kafka.connect.runtime.WorkerSourceTask.execute(WorkerSourceTask.java:155)
        at org.apache.kafka.connect.runtime.WorkerTask.doRun(WorkerTask.java:140)
        at org.apache.kafka.connect.runtime.WorkerTask.run(WorkerTask.java:175)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:853)
``` 

向store发送数据的堆栈: 

```
ts=2023-11-08 14:37:40;thread_name=pool-2-thread-1;id=2f;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@764c12b6
    @org.apache.kafka.connect.runtime.drcdeliver.DRCDeliverSourceWorker.sendRecords()
        at org.apache.kafka.connect.runtime.WorkerSourceTask.execute(WorkerSourceTask.java:160)
        at org.apache.kafka.connect.runtime.WorkerTask.doRun(WorkerTask.java:140)
        at org.apache.kafka.connect.runtime.WorkerTask.run(WorkerTask.java:175)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:853)
``` 

梳理源端的代码路径: 

  - Kafka connect 框架
    - source端获取数据: [Kafka connect负责调度线程]: LogMinerConnectorTask.poll() 
      - LogMinerConnectorTask 获取数据: [TaskRecordQueue线程, 线程名"Connector Task Record Queue", 由LogMinerConnectorTask生成]: com.taobao.drc.logminer.connect.LogMinerConnectorTask.TaskRecordQueue#run
        - 获取数据的来源是: 全局静态: com.taobao.drc.logminer.LogminerWrapper#scnIncrementRecordQueue
    - target写入数据: [Kafka connect负责调度线程]: DRCDeliverSourceWorker.sendRecords()
      - 往store中写入

  - scnIncrementRecordQueue的数据写入流程
    - 线程: com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSorter#run, 由com.taobao.drc.logminer.queue.OracleBackRecordQueue生成  

      - com.taobao.drc.logminer.queue.OracleBackRecordQueue, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator生成
    - 线程: com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordDispatcher#run, 由com.taobao.drc.logminer.queue.OracleBackRecordQueue生成  

      - com.taobao.drc.logminer.queue.OracleBackRecordQueue, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator生成
    - 线程: com.taobao.drc.logminer.LogRecordConverterCore#run, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper生成  

      - com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator生成
    - 线程: com.taobao.drc.logminer.LogRecordConverterPrepare#run, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper生成  

      - com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator生成
    - 线程: com.taobao.drc.logminer.LogRecordSerializer#run, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper生成
      - com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator生成
    - 线程: com.taobao.drc.logminer.OracleLogExtractor#run, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator生成
    - 线程: com.taobao.drc.logminer.fetcher.LogEntryFetcher#run, 由com.taobao.drc.logminer.OracleRacLogRecordGenerator生成

### TODO

store还有一根连接是从supervisor来的, 应该是探活连接?

# 结论

store组件, 是一个容量极大的队列, 无法将其填满, 并且其从日志中, 不能暴露自己积压了多少内容

所以阻塞目标端时, 源端将数据全部转移到了store中.

通过gdb阻塞store, 可以观察到源端队列的积压: 

```
bash-4.2$ bash get_status.sh
    @String[fetcherOutQueueSize=20000],
    @String[extractorOutQueueSize=20000],
    @String[logAggregatorOutQueueSize=20000],
    @String[preparedRecordOutQueueSize=20000],
    @String[logRecordConverterOutQueueSize=20000],
    @String[dispatchOutIdsSize=20000],
    @String[dispatchOutRecordsSize=20001],
    @String[dispatchOutQueueSize=0],
    @String[scnIncrementRecordQueueInSize=20000],
    @String[sourceRecordQueueSize=20000],
    @String[sourceOutQueueSize=0],
    @String[bridgeThread0OutQueueSize=0],
    @String[bridgeThread1OutQueueSize=0],
    @String[sinkTasksCount=64],
``` 

TODO: dispatchOutQueueSize 始终为0, 需要确认问题
