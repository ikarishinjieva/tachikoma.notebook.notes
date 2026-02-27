---
title: 20231108 - 整理OMS的增量复制链路
confluence_page_id: 2589408
created_at: 2023-11-08T15:31:43+00:00
updated_at: 2023-11-28T08:25:32+00:00
---

# 目录

# 起点: 源数据库

Oracle

# 1: Logminer订阅源数据库变更 (订阅redo log) 

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: LogEntryPrimaryFetcher#1 (RAC模式下, 会有多个)

线程主函数: 

```
com.taobao.drc.logminer.fetcher.LogEntryFetcher#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.fetcher.LogEntryFetcher 
``` 

  

数据来源: 读取LogMiner的逻辑位置: 

```
at com.taobao.drc.logminer.fetcher.LogMiner.loadLogEntries(LogMiner.java:172)
at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithValidationExceptionResume(LogMinerUtil.java:254)
at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithValidationExceptionResume(LogMinerUtil.java:222)
at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithBreakpointResume(LogMinerUtil.java:207)
at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadRedoLogEntries(LogMinerUtil.java:148)
at com.taobao.drc.logminer.fetcher.LogEntryPrimaryFetcher.fetchLogEntriesFromRedoLogFile(LogEntryPrimaryFetcher.java:67)
at com.taobao.drc.logminer.fetcher.LogEntryPrimaryFetcher.fetchNonArchLogEntriesOneRound(LogEntryPrimaryFetcher.java:59)
at com.taobao.drc.logminer.fetcher.LogEntryFetcher.notOnlyFetchArchLogEntriesLoop(LogEntryFetcher.java:147)
at com.taobao.drc.logminer.fetcher.LogEntryFetcher.fetchLogEntriesLoop(LogEntryFetcher.java:116)
at com.taobao.drc.logminer.fetcher.LogEntryFetcher.run(LogEntryFetcher.java:91)
``` 

  

数据去向: 队列命名为 LogEntryFetcher_OutQueue

```
在com.taobao.drc.logminer.fetcher.LogMiner.loadLogEntries中, 
通过
logEntryConsumer.acceptWithInterruptionPreCheck(logEntry)
调用
com.taobao.drc.logminer.fetcher.LogEntryFetcher#putEntry
写入队列
fetcherOutQueue
``` 

  

    
    
    队列元素: com.taobao.drc.logminer.log.OracleLogEntry

详细说明: [20231115 - Oracle logminer学习]
    
    
     

# 2: 对redo log项 进行分析

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Oracle Log Extractor Instance#1 (RAC模式下, 会有多个)

线程主函数: 

```
com.taobao.drc.logminer.OracleLogExtractor#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.OracleLogExtractor
``` 

数据来源: 上一步中的 LogEntryFetcher_OutQueue

数据去向: 队列命名为 OracleLogExtractor_OutQueue

```
在com.taobao.drc.logminer.OracleLogExtractor#run 中, 通过 this.out.putLogRecord(record) 输出到队列
``` 
    
    
    队列元素: com.taobao.drc.logminer.log.OracleLogRecord

该步骤的大致处理逻辑: 

  1. 将获取的 redo log项 进行分配, 分配给4个分析线程 (LogminerWrapper.ANALYSER_THREAD_NUM_VALUE)
     1. 每分析线程每批次最多分配到1000个 redo log项 (LogminerConfig.DEFAULT_ANALYSER_BATCH_SIZE)
     2. 需要遵循的原则是: 连续的redo log项 (CSF != 0), 不可被拆分
  2. 使用4个分析线程, 进行分析
     1. 使用不同的分析器 (如下图), 将redo log项 转成 com.taobao.drc.logminer.log.OracleLogRecord 结构
  3. 等待4个分析线程分析完成, 依序取出结果, 依次给到输出队列

  

FIXME: 

  - 4线程一批, 但依序取出, 会导致分析效率降低. 最差情况: 后三个线程任务结束进入空闲等待, 但第一线程任务阻塞

  

# 3: 识别并填写事务信息

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Oracle LogRecord Serializer Instance#1 (RAC模式下, 会有多个)

线程主函数: 

```
com.taobao.drc.logminer.LogRecordSerializer#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper
------------> com.taobao.drc.logminer.LogRecordSerializer
``` 

数据来源: 上一步中的 OracleLogExtractor_OutQueue

数据去向: 队列命名为 LogRecordSerializer_OutQueue

```
在 com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator#dealCommittedTransaction, 处理COMMIT记录时, 将整个事务的数据记录 通过enqueue写入队列
``` 

  

队列元素: com.taobao.drc.logminer.log.OracleLogRecord

该步骤的大致处理逻辑: 

  1. 逐一处理数据, 
     1. 还未到事务结束 (COMMIT) 时, 将数据暂存 (保存到com.oceanbase.xlog.util.BigList, 其中有文件存储)
     2. 识别到COMMIT时, 将事务信息提取出来, 填写到事务中的所有数据记录中, 并将所有暂存数据送到下个队列

  

  

# 4: 将之前步骤中, RAC模式的多个流排序成一个流

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Oracle LogRecord Converter Prepare (只会有一个)

线程主函数: 

```
com.taobao.drc.logminer.LogRecordConverterPrepare#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper
------------> com.taobao.drc.logminer.LogRecordConverterPrepare
``` 

数据来源: 上一步中的 LogRecordSerializer_OutQueue (多个)

数据去向: 队列命名为 LogRecordConverterPrepare_OutQueue (一个)

```
在 com.taobao.drc.logminer.LogRecordConverterPrepare#prepareConvertLogRecordsForTrx, 写入队列
``` 

队列元素: com.taobao.drc.logminer.log.OracleLogRecord

该步骤的大致处理逻辑: 

  1. 从每个流中拉取最小的SCN
  2. 将 最小的SCN对应的事务中的数据 写入输出队列, 然后进行下一轮

  

FIXME:

  1. 如果一个流活跃, 一个流不活跃 (不确定RAC是否会出现这个情况), 那么此步骤会一直阻塞, 直到能从不活跃的流中获取数据

  

# 5: 将数据转换成 IncrementRecord 格式

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Oracle LogRecord Converter Core (只会有一个)

线程主函数: 

```
com.taobao.drc.logminer.LogRecordConverterCore#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.OracleRacLogRecordGenerator.SerializersConverterWrapper
------------> com.taobao.drc.logminer.LogRecordConverterCore
``` 

数据来源: 上一步中的 LogRecordConverterPrepare_OutQueue (一个)

数据去向: 队列命名为 LogRecordConverterCore_OutQueue (一个)

```
在 com.taobao.drc.logminer.LogRecordConverterCore#convertRecordsCore, 写入队列
``` 

队列元素: com.taobao.drc.logminer.io.IncrementRecord

该步骤的大致处理逻辑: 

  1. 并发8个线程, 进行格式转换 (LogminerWrapper.CONVERTER_THREAD_NUM_VALUE). 格式转换的好处不明?
     1. 以8线程为例, 并发每批次下发8个任务, 等待8个任务完全完成, 才会进行下个批次 (FIXME: 效率低下, com.taobao.drc.logminer.LogRecordConverterCore#run)
  2. 主要转换逻辑在 com.taobao.drc.logminer.OracleLogBackConverter 中
     1. 增加ROWID伪列?
     2. 以DDL为界, DDL前的数据必需回放完, 在DDL处理时更新元数据, DDL后的数据必须在DDL更新完后回放
        1. 分界逻辑在 com.taobao.drc.logminer.LogRecordConverterCore#convertRecord (FIXME: 使用递归形式, 代码复杂)

# 6: 将需要回表查询的记录, 和不需要回表的记录, 进行分流

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Record Dispatcher (只会有一个)

线程主函数: 

```
com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordDispatcher#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.queue.OracleBackRecordQueue
------------> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordDispatcher
``` 

数据来源: 上一步中的 LogRecordConverterCore_OutQueue (一个)

数据去向: 两个队列: 

  - 队列命名为 RecordDispatcher_OutQueue_Ids (一个), 存放所有记录的ID (com.taobao.drc.logminer.queue.OracleBackRecordQueue#ids) (记录的数据保存在com.taobao.drc.logminer.queue.OracleBackRecordQueue#records中, 供查询)
  - 队列命名为 RecordDispatcher_OutQueue (一个), 存放需要回表的记录 (com.taobao.drc.logminer.queue.OracleBackRecordQueue#queue)  
  

队列元素: 

  - RecordDispatcher_OutQueue_Ids: Long  

  - RecordDispatcher_OutQueue: [com.taobao.drc.logminer.io](<http://com.taobao.drc.logminer.io>).SCNIncrementRecord

  

# 7.1: 回表查询

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Record Selector-xxx (默认32个, LogminerWrapper.SELECTOR_THREAD_NUM_VALUE)

线程主函数: 

```
com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSelector#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.queue.OracleBackRecordQueue
------------> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSelector
``` 

数据来源: 上一步中的 RecordDispatcher_OutQueue (一个)

数据去向: 存放到上一步的com.taobao.drc.logminer.queue.OracleBackRecordQueue#records结构中, 供查询

  

# 7.2: 将回表/不回表的数据 进行串行合并, 并进行数据修整

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Record Sorter (只有一个)

线程主函数: 

```
com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSorter#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.LogminerWrapper#start
------> com.taobao.drc.logminer.OracleRacLogRecordGenerator
---------> com.taobao.drc.logminer.queue.OracleBackRecordQueue
------------> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSorter
``` 

数据来源: 上一步中的 RecordDispatcher_OutQueue_Ids 记录ID, 在上一步中的com.taobao.drc.logminer.queue.OracleBackRecordQueue#records结构查询数据

数据去向: LogminerWrapper.scnIncrementRecordQueue (FIXME: 全局静态队列)

```
com.taobao.drc.logminer.queue.OracleBackRecordQueue#put 写入队列
``` 

队列元素: com.taobao.drc.logminer.io.SCNIncrementRecord

该步骤的大致处理逻辑: 

  1. 从RecordDispatcher_OutQueue_Ids依次取出ID, 完成数据串行化. 如果记录 未完成回表, 则等待回表完成. 
  2. 对于必要情况, 增加心跳数据包 (com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSorter#run)
  3. 对于LogSCNMultiIncrementRecord, 分拆数据包

# 8: 将数据格式转换为org.apache.kafka.connect.source.SourceRecord

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: Connector Task Record Queue (只有一个)

线程主函数: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask.TaskRecordQueue#run
``` 

线程来源: 

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
---> com.taobao.drc.logminer.connect.LogMinerConnectorTask.TaskRecordQueue
``` 

数据来源: LogminerWrapper.scnIncrementRecordQueue (FIXME: 全局静态队列)

数据去向: 队列命名为 TaskRecordQueue_OutQueue

```
com.taobao.drc.logminer.connect.LogMinerConnectorTask.TaskRecordQueue#run 写入队列
``` 

队列元素: org.apache.kafka.connect.source.SourceRecord

# 9: 发送数据到store 

进程: 源端进程(org.apache.kafka.connect.cli.ConnectDRCDeliver)

线程名: pool-2-thread-1 (kafka connect线程, 不确定数量限制, 配置tasks.max未配置, 默认值是1)

线程来源:

```
进程main: org.apache.kafka.connect.cli.ConnectDRCDeliver
---> 将 org.apache.kafka.connect.runtime.drcdeliver.DRCDeliverWorkerSourceTask 注册到 kafka connect
------> org.apache.kafka.connect.runtime.drcdeliver.DRCDeliverWorkerSourceTask#buildWorkerTask 生成 org.apache.kafka.connect.runtime.drcdeliver.DRCDeliverSourceWorker
``` 

数据来源: 上一步骤的TaskRecordQueue_OutQueue

```
kafka connect框架, 通过com.taobao.drc.logminer.connect.LogMinerConnectorTask#poll, 从TaskRecordQueue_OutQueue获取数据
``` 

数据去向: 发往store进程

```
org.apache.kafka.connect.runtime.drcdeliver.DRCDeliverSourceWorker#sendRecords, 将数据通过网络发给store进程
``` 

  

特别说明: 

Kafka connect框架的生效范围在此步骤中, 框架负责 管理 LogMinerConnectorTask 和 DRCDeliverSourceWorker. 

之前各步骤的线程, 均是 LogMinerConnectorTask 负责生成和管理. 

对于为什么要用Kafka connect框架, 意义不明 (未见扩展性或者高可用性的好处, 仅在处理该步骤的checkpoint时有好处, 但不多)

# 附录: com.taobao.drc.logminer.connect.LogMinerConnectorTask的来源

```
进程main: org.apache.kafka.connect.cli.ConnectDRCDeliver
---> org.apache.kafka.connect.cli.ConnectDRCDeliver#startConnector 通过配置 ./conf/deliver2store.conf的配置, 创建Connector
		(org.apache.kafka.connect.runtime.drcdeliver.DRCDeliverHerder#putConnectorConfig 会调用 org.apache.kafka.connect.runtime.drcdeliver.DRCDeliverHerder#startConnector)
------> com.taobao.drc.logminer.connect.LogMinerConnector#taskClass, 获取Task的类
---------> com.taobao.drc.logminer.connect.LogMinerConnectorTask#start
``` 

# 10: store (相当于带有持久化功能的消息队列)

  

# 11: 从store中接收数据, 拼接/切割成事务

进程: 目标端进程(com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap)

线程名: DRC-Client-Thread (DRC通讯客户端)

线程来源:

```
com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap#main
---> com.oceanbase.oms.connector.coordinator.BootStrapPanel#boot
------> sourceBooster.start()
---------> com.oceanbase.oms.connector.source.store.StoreSource#start
------------> com.oceanbase.oms.store.client.impl.DRCClientImpl#startService
---------------> com.oceanbase.oms.store.client.impl.DRCClientImpl.run
``` 

数据来源: store, 通过DRC API获取

数据去向: 队列命名为 StoreSource_OutQueue

```
@com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction()
        at com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.generateTransaction(CommonTransactionAssembler.java:228)
        at com.oceanbase.oms.connector.batch.CommonTransactionAssembler.notify(CommonTransactionAssembler.java:105)
        at com.oceanbase.oms.connector.source.store.TransactionAssembler.notify(TransactionAssembler.java:96)
        at com.oceanbase.oms.store.client.impl.DRCClientImpl.notifyDataMessage(DRCClientImpl.java:568)
        at com.oceanbase.oms.store.client.impl.DRCClientImpl.notifyDataMessage(DRCClientImpl.java:534)
        at com.oceanbase.oms.store.client.impl.DRCClientImpl.run(DRCClientImpl.java:451)
        at java.lang.Thread.run(Thread.java:853)
``` 

队列元素: com.oceanbase.oms.connector.common.transaction.Transaction

  

特别地: 此处有一个事务切割器: 

```
事务切割器: com.oceanbase.oms.connector.batch.CommonTransactionAssembler.TransactionAssemblerInner#transactionCutter
 
构建位置: com.oceanbase.oms.connector.source.store.StoreSource#buildRecordListener
  
配置:
source.splitThreshold, 事务切分的阈值, 默认值128个数据
source.sourceBatchMemorySize, 事务切分的内存阈值, 默认值为-1
``` 

事务切割器会将原始事务切割成小事务. 需要注意: 

  1. 如果切割得太小, 下游的计算和处理成本 高于 收益
  2. 如果切割得太大, 会影响到下游事务并发的收益

  

# 12: ETL 数据转换规则

进程: 目标端进程(com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap)

线程名: forward_slot0-(ETLProcessor)-queue_slot1

说明: 本线程由BridgeTaskManager管理, BridgeTaskManager会根据需求定义 数据处理链, 本线程是 Oracle->OB 的增量复制 环节的处理链的第一个环节, 以此为例 (其他处理链情况尚不清楚)

线程来源:

```
com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap#main
---> com.oceanbase.oms.connector.coordinator.BootStrapPanel#boot
------> com.oceanbase.connector.framework.threadmanager.TaskManagerAllocator#buildTaskManagers
------> bridgeTaskManager.start()
``` 

数据来源: 上一步骤中的 StoreSource_OutQueue

数据去向: 

  - 队列命名为 ETLProcessor_OutQueue

```
数据处理堆栈: 
    @com.oceanbase.oms.connector.customize.batchprocessor.ETLBatchProcessor.process()
        at com.oceanbase.connector.framework.threadmanager.bridgetask.SerialBridgeConnectorTask.processRecordBatch(SerialBridgeConnectorTask.java:105)
        at com.oceanbase.connector.framework.threadmanager.bridgetask.SerialBridgeConnectorTask.run(SerialBridgeConnectorTask.java:79)
        at java.lang.Thread.run(Thread.java:853)
 
数据处理后在SerialBridgeConnectorTask中写入输出队列
```

队列元素: com.oceanbase.oms.connector.common.transaction.Transaction

该步骤的大致处理逻辑: 

  - etlBatchProcessor: 
    - ddlTransformerChain
      - DDLTypeDbTableFilterTransformer
        - 根据黑白名单配置, 剔除不需要迁移的DDL
      - DDLTransformer
        - 将 源数据库 的DDL, 转换成 目标数据库的DDL (使用中间的DDL解析结构 进行 中介)
    - dmlTransformerChain
      - DMLTypeFilterTransformer
        - 根据黑白名单配置, 剔除不需要迁移的DML
      - RowFilterTransformer
        - 根据黑白名单配置, 剔除不需要迁移的 行
      - ColumnFilterTransformer
        - 根据黑白名单配置, 剔除不需要迁移的 列

# 13: 梳理事务顺序

进程: 目标端进程(com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap)

线程名: queue_slot1-()-TransactionScheduler

说明: 本线程由BridgeTaskManager管理, BridgeTaskManager会根据需求定义 数据处理链, 本线程是 Oracle->OB 的增量复制 环节的处理链的第二个环节

线程来源:

```
com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap#main
---> com.oceanbase.oms.connector.coordinator.BootStrapPanel#boot
------> com.oceanbase.connector.framework.threadmanager.TaskManagerAllocator#buildTaskManagers
------> bridgeTaskManager.start()
``` 

数据来源: 上一步骤中的 ETLProcessor_OutQueue

```
数据处理堆栈: 
    @com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler.offer()
        at com.oceanbase.connector.framework.threadmanager.bridgetask.SerialBridgeConnectorTask.run(SerialBridgeConnectorTask.java:86)
        at java.lang.Thread.run(Thread.java:853)
``` 

  

  

数据去向: 队列命名为 TransactionScheduler_OutQueue
    
    
    队列元素: com.oceanbase.oms.connector.common.transaction.Transaction

该步骤的大致处理逻辑: 

  - transactionScheduler, 梳理事务顺序. (DefaultTransactionScheduler)有三个作用: 
    - 限流阀: 
      - 阀内能容下 maxRecordCapacity 个行, 超过则等待消费
      - 限流阀: 命名为: TransactionScheduler_In_Rows / TransactionScheduler_In_MaxRows

        - 维护了一个 用于内存限制 的准入机制: 
          - 在事务 写入 处理器TransactionScheduler 时, 将事务中的数据行数 累加到 数据计数
            1. 当 数据计数达到限制 时, 进行准入等待
            2. 限制配置项为: maxRecordCapacity, 默认值: 16384
          - 当事务被 处理器Sink消费完成 (已经完全回放到目标数据库中)时, 将事务中的数据行数 从数据计数中 去除
          - 此处内存限制, 也意味着 依赖管理图 中的事务的数量, 
            - 也等于 因为依赖原因还未提交的事务 + 因为连接饱和还未提交的事务 + 在连接中正在提交的事务
            - 在连接中正在提交的事务 = sink的活跃数
            - 因为连接饱和还未提交的事务 = 该阶段的输出队列积压数
            - 因为依赖原因还未提交的事务 = 依赖管理图 中的事务的数量 - 在连接中正在提交的事务 - 因为连接饱和还未提交的事务
            - FIXME: 此处统计值可优化
    - (checkpointManager) 生成checkpoint: 
      - 当阀内的 事务 完全消费掉, 生成一个checkpoint
    - (conflictBroker) 依赖管理:
      - 根据DML事务的行的唯一键值, 计算事务的依赖图 (拓扑顺序)
      - ConflictBrokerV1 只计算依赖, 事务进入DefaultTransactionScheduler后, 就进入ConflictBrokerV1计算依赖. 当其依赖的前序事务都确认提交后, 该事务从DefaultTransactionScheduler输出

  

# 14: 向目标数据库回放数据

进程: 目标端进程(com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap)

线程名: sinkTask-1 (默认16个, 通过sink.workerNum配置)

说明: 本组线程由SinkTaskManager管理

线程来源:

```
com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap#main
---> com.oceanbase.oms.connector.coordinator.BootStrapPanel#boot
------> com.oceanbase.connector.framework.threadmanager.TaskManagerAllocator#buildTaskManagers
------> sinkTaskManager.start()
``` 

数据来源: 上一步骤中的 TransactionScheduler_OutQueue

数据去向: 通过JDBC写入目标数据库

```
数据去向堆栈: 
    @com.oceanbase.oms.connector.jdbc.sink.Writer.flushRecords()
        at com.oceanbase.oms.connector.jdbc.sink.DefaultJDBCSink.offer(DefaultJDBCSink.java:60)
        at com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer(OBOracleJDBCSink.java:25)
        at com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask.run(SyncSinkConnectorTask.java:44)
        at java.lang.Thread.run(Thread.java:853)
``` 

# 15: 目标数据库
