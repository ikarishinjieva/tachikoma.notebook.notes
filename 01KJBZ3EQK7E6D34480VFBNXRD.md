---
title: 20231025 - 通过arthas获取OMS队列监控
confluence_page_id: 2589267
created_at: 2023-10-25T08:35:21+00:00
updated_at: 2023-10-26T12:29:39+00:00
---

# 连接信息

```
服务器: 
10.186.16.126

容器: OMS_20231025_122226

UI:
http://10.186.16.126:8089 
admin/@aaAA11__

源：
 ip: 10.186.16.126:1521
 user: obtest
 password: 111111
 service name: EE.ORACLE.DOCKER

目标：
 ip: 10.186.16.126:32883
 user: obtest@oms_oracle
 password: 111111
 集群名: obthree

meta元数据：
 ip: 10.186.16.126:32883
 user: root
 password: oms

ob-oracle 目标端:
  obclient -h10.186.16.126 -P32883 -uobtest@oms_oracle -p111111 -A

oracle 源端: (oracle部署在126容器内)
  docker exec -it relaxed_wu bash
  su oracle
  sqlplus obtest/111111 
  sqlplus / as sysdba
``` 

插入数据: 

select * from OBTEST1.T1;  
insert into OBTEST1.T1(ID, FIRST_NAME) values(6, 'ccccc');

# 获取jar

复制出所有jar

```
ps aux | grep [j]ava | awk '{print $2}' | xargs -L1 -I{} ls -alh /proc/{}/fd | grep '.jar' | grep -v 'jre' | grep -v 'dragonwell' | awk '{print $NF}' | xargs -L1 -I{} cp {} /tmp/jars
``` 

# Arthas使用举例

```
[root@R740-26 arthas]# su ds
 
bash-4.2$ java -jar arthas-boot.jar
...
[arthas@67578]$ vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleLogEntryQueue --limit 10
@OracleLogEntryQueue[][
    @OracleLogEntryQueue[com.taobao.drc.logminer.queue.OracleLogEntryQueue@57040d31],
]
 
[arthas@67578]$ vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleLogEntryQueue --limit 2 --express 'instances[0].size()'
vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleLogEntryQueue --limit 2 --express 'instances[0].size()'
@Integer[0]
 
 
``` 

# 初步整理监控项

### 进程: org.apache.kafka.connect.cli.ConnectDRCDeliver

监控项:

```
vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleLogEntryQueue --limit 2 --express 'instances[0].size()'

vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleLogRecordQueue --limit 2 --express 'instances[0].pending()'

vmtool --action getInstances --className com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator --limit 2 --express 'instances[0].pending()'
 
# preparedRecordQueue/preparedConvertedRecordQueue
vmtool --action getInstances --className com.taobao.drc.logminer.LogRecordConverterPrepare --limit 2 --express 'instances[0].getQueueSize()'
 
# OracleBackRecordQueue.in
vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleBackRecordQueue --limit 2 --express 'instances[0].pending()'
 
# OracleBackRecordQueue.queue
vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleBackRecordQueue --limit 2 --express 'instances[0].queue.size()'
 
# OracleBackRecordQueue.records
vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleBackRecordQueue --limit 2 --express 'instances[0].records.size()'
 
# LogminerWrapper.scnIncrementRecordQueue
vmtool --action getInstances --className com.taobao.drc.logminer.queue.OracleSCNIncrementRecordQueue --limit 2 --express 'instances[0].pending()'
 
# TaskRecordQueue#records
vmtool --action getInstances --className com.taobao.drc.logminer.connect.LogMinerConnectorTask --limit 2 --express 'instances[0].taskRecordQueue.getTaskQueueSize()'
``` 

### 进程: com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap

```
# storeRecordListener -> forward_slot
vmtool --action getInstances --className com.oceanbase.oms.connector.batch.CommonTransactionAssembler --limit 2 --express 'instances[0].transactionAssemblerInner.sink.sink.getAccumulatedBatchSize()'
 
# BridgeTaskManager(ETLProcessor) -> queue_slot
vmtool --action getInstances --className com.oceanbase.connector.framework.threadmanager.bridgetask.BridgeTaskManager --limit 2 --express 'instances[0].threads.toArray()[0].sink.getAccumulatedBatchSize()'
 
# BridgeTaskManager(TransactionScheduler) -> queue_slot
vmtool --action getInstances --className com.oceanbase.connector.framework.threadmanager.bridgetask.BridgeTaskManager --limit 2 --express 'instances[0].threads.toArray()[1].sink.getAccumulatedBatchSize()'
 
 
# 活跃的sink连接?
vmtool --action getInstances --className com.oceanbase.oms.connector.jdbc.sink.Writer --limit 100 --express 'instances.{? #this.connection != null}'
``` 

# 再次整理

进程: org.apache.kafka.connect.cli.ConnectDRCDeliver

```
# 按照数据库url+username, 找到对应的OracleRacLogRecordGenerator, 从Generator找到各队列 (但到LogminerWrapper.scnIncrementRecordQueue结束)
 
vmtool --action getInstances --className com.taobao.drc.logminer.OracleRacLogRecordGenerator --limit 100 --express '#generator=instances.{? #this.context.logmnrDataSource.url == "10.186.16.126:1521/EE.ORACLE.DOCKER" && #this.context.logmnrDataSource.username == "OBTEST"}[0],
#checkpointCount=#generator.context.ctsCheckpoints.size(),
#checkpointCount == 1 ? "assertion" : #assert_fail[444],
#logThreadNumber=#generator.context.ctsCheckpoints[0].threadNumber,
"------",
"com.taobao.drc.logminer.fetcher.LogEntryFetcher#putEntry -> fetcherOutQueue",
#generator.fetchers.size() == 1 ? "assertion" : #assert_fail[444],
#fetcher=#generator.fetchers[0],
#fetcherOutQueueSize=#fetcher.getQueueSize(),
"------",
"fetcherOutQueue -> com.taobao.drc.logminer.OracleLogExtractor#run -> extractorOutQueue",
#generator.extractors.size() == 1 ? "assertion" : #assert_fail[444],
#extractor=#generator.extractors[0],
#extractor["in"] == #fetcher.queue ? "assertion" : #assert_fail[444],
#extractorOutQueueSize=#extractor.getQueueSize(),
"------",
"extractorOutQueue -> com.taobao.drc.logminer.LogRecordSerializer#run -> logAggregatorOutQueue",
#generator.serializersConverterWrapper.serializers.size() == 1 ? "assertion" : #assert_fail[444],
#serializer=#generator.serializersConverterWrapper.serializers[0],
#serializer["in"] == #extractor.out ? "assertion" : #assert_fail[444],
#logAggregatorOutQueue=#serializer.out,
#logAggregatorOutQueueSize=#logAggregatorOutQueue.pending(),
"------",
"logAggregatorOutQueue -> com.taobao.drc.logminer.LogRecordConverterPrepare#run -> preparedRecordOutQueue",
#converterPrepare = #generator.serializersConverterWrapper.converterPrepare,
#converterPrepare.logAggregators.size() == 1 ? "assertion" : #assert_fail[444],
#converterPrepare.logAggregators[0] == #logAggregatorOutQueue ? "assertion" : #assert_fail[444],
#preparedRecordOutQueue=#converterPrepare.preparedRecordQueue,
#preparedRecordOutQueueSize=#preparedRecordOutQueue.size(),
"------",
"preparedRecordOutQueue -> com.taobao.drc.logminer.LogRecordConverterCore#run -> logRecordConverterOutQueue",
#converterCore = #generator.serializersConverterWrapper.converterCore,
#converterCore.inQueue == #preparedRecordOutQueue ? "assertion" : #assert_fail[444],
#logRecordConverterOutQueue = #converterCore.outQueue,
#logRecordConverterOutQueueSize=#logRecordConverterOutQueue["in"].size(),
"------",
"logRecordConverterOutQueue -> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordDispatcher#run -> dispatchOutIds, dispatchOutRecords, dispatchOutQueue,",
"dispatchOutQueue -> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSelector#run -> dispatchOutRecords",
#logRecordConverterOutQueue == #generator.queue ? "assertion" : #assert_fail[444],
#dispatchOutIds = #generator.queue.ids,
#dispatchOutRecords = #generator.queue.records,
#dispatchOutQueue = #generator.queue.queue,
#dispatchOutIdsSize = #dispatchOutIds.size(),
#dispatchOutRecordsSize = #dispatchOutRecords.size(),
#dispatchOutQueueSize = #dispatchOutQueue.size(),
{#dispatchOutIdsSize, #dispatchOutRecordsSize, #dispatchOutQueueSize},
"------",
"dispatchOutIds,dispatchOutRecords -> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSorter#run -> LogminerWrapper.scnIncrementRecordQueue",
#scnIncrementRecordQueueInSize = #generator.queue.getAfterSelectQueueSize(),
"------",
"result:", 
{
"fetcherOutQueueSize=" + #fetcherOutQueueSize, 
"extractorOutQueueSize=" + #extractorOutQueueSize, 
"logAggregatorOutQueueSize=" + #logAggregatorOutQueueSize, 
"preparedRecordOutQueueSize=" + #preparedRecordOutQueueSize, 
"logRecordConverterOutQueueSize=" + #logRecordConverterOutQueueSize, 
"dispatchOutIdsSize=" + #dispatchOutIdsSize, 
"dispatchOutRecordsSize=" + #dispatchOutRecordsSize, 
"dispatchOutQueueSize=" + #dispatchOutQueueSize, 
"scnIncrementRecordQueueInSize=" + #scnIncrementRecordQueueInSize}
'
 
# 从LogminerWrapper.scnIncrementRecordQueue开始梳理队列, 直到Kafka
 
vmtool --action getInstances --className com.taobao.drc.logminer.connect.LogMinerConnectorTask --express '
"------",
"LogminerWrapper.scnIncrementRecordQueue -> com.taobao.drc.logminer.connect.LogMinerConnectorTask.TaskRecordQueue#run -> sourceRecordQueue",
#sourceRecordQueueSize = instances[0].taskRecordQueue.records.size(),
"------",
"sourceRecordQueue -> com.taobao.drc.logminer.connect.LogMinerConnectorTask.poll() -> kafka(org.apache.kafka.connect.runtime.WorkerSourceTask.execute)",
"kafa configuration in /u01/ds/store/store7100/conf/deliver2store.conf",
"------",
{"sourceRecordQueueSize="+#sourceRecordQueueSize}
'
``` 

进程: com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap

```
vmtool --action getInstances --className com.oceanbase.connector.framework.CoordinatorContext --express '
#coodinator=instances[0],
#sourceConfig=#coodinator.sourceContext.config.sectionKeyValueMap["source"]["clients"],
"TODO: check config to match source",
"------",
"store RPC -> DRCClientImpl.notifyDataMessage -> com.oceanbase.oms.connector.source.store.TransactionAssembler.notify -> com.oceanbase.oms.connector.batch.CommonTransactionAssembler.notify -> sourceOutQueue",
#sourceOutQueue=#coodinator.sourceTaskManager.sink,
#sourceOutQueueSize=#sourceOutQueue.getAccumulatedBatchSize(),
"------",
"sourceOutQueue -> bridge thread[0](ETLProcessor) -> bridgeThread0OutQueue",
#bridgeThreads=#coodinator.bridgeTaskManager.threads.toArray(),
#bridgeThreads.length == 2 ? "assertion" : #assert_fail[444],
#bridgeThread0=#bridgeThreads[0],
#bridgeThread0Name=#bridgeThread0.threadName,
#bridgeThread0.source ==  #sourceOutQueue ? "assertion" : #assert_fail[444],
#bridgeThread0OutQueue=#bridgeThread0.sink,
#bridgeThread0OutQueueSize=#bridgeThread0OutQueue.getAccumulatedBatchSize(),
"------",
"bridgeThread0OutQueue -> bridge thread[1](TransactionScheduler) -> bridgeThread1OutQueue",
#bridgeThread1=#bridgeThreads[1],
#bridgeThread1Name=#bridgeThread1.threadName,
#bridgeThread1.source ==  #bridgeThread0OutQueue ? "assertion" : #assert_fail[444],
#bridgeThread1OutQueue=#bridgeThread1.sink,
#bridgeThread1OutQueueSize=#bridgeThread1OutQueue.getAccumulatedBatchSize(),
"------",
"bridgeThread1OutQueue -> n * com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask#run -> OBOracleJDBCSink",
#coodinator.sinkTaskManager.source == #bridgeThread1OutQueue ?  "assertion" : #assert_fail[444],
#sinkTasks=#coodinator.sinkTaskManager.threads,
#sinkTasksCount=#sinkTasks.size(),
#sinkTasks.toArray()[0].threadName,
"TODO: how to get running-status and current sql of DefaultJDBCSink",
"------",
{"sourceOutQueueSize=" + #sourceOutQueueSize, 
"bridgeThread0OutQueueSize=" + #bridgeThread0OutQueueSize, 
"bridgeThread1OutQueueSize=" + #bridgeThread1OutQueueSize, 
"sinkTasksCount=" + #sinkTasksCount}
'
``` 

写成统一脚本: (as脚本, 不允许单一命令的换行, 需要删除换行)

```
bash-4.2$ cat 1.as
vmtool --action getInstances --className com.taobao.drc.logminer.OracleRacLogRecordGenerator --limit 100 --express '#generator=instances.{? #this.context.logmnrDataSource.url == "10.186.16.126:1521/EE.ORACLE.DOCKER" && #this.context.logmnrDataSource.username == "OBTEST"}[0],#checkpointCount=#generator.context.ctsCheckpoints.size(),#checkpointCount == 1 ? "assertion" : #assert_fail[444],#logThreadNumber=#generator.context.ctsCheckpoints[0].threadNumber,"------","com.taobao.drc.logminer.fetcher.LogEntryFetcher#putEntry -> fetcherOutQueue",#generator.fetchers.size() == 1 ? "assertion" : #assert_fail[444],#fetcher=#generator.fetchers[0],#fetcherOutQueueSize=#fetcher.getQueueSize(),"------","fetcherOutQueue -> com.taobao.drc.logminer.OracleLogExtractor#run -> extractorOutQueue",#generator.extractors.size() == 1 ? "assertion" : #assert_fail[444],#extractor=#generator.extractors[0],#extractor["in"] == #fetcher.queue ? "assertion" : #assert_fail[444],#extractorOutQueueSize=#extractor.getQueueSize(),"------","extractorOutQueue -> com.taobao.drc.logminer.LogRecordSerializer#run -> logAggregatorOutQueue",#generator.serializersConverterWrapper.serializers.size() == 1 ? "assertion" : #assert_fail[444],#serializer=#generator.serializersConverterWrapper.serializers[0],#serializer["in"] == #extractor.out ? "assertion" : #assert_fail[444],#logAggregatorOutQueue=#serializer.out,#logAggregatorOutQueueSize=#logAggregatorOutQueue.pending(),"------","logAggregatorOutQueue -> com.taobao.drc.logminer.LogRecordConverterPrepare#run -> preparedRecordOutQueue",#converterPrepare = #generator.serializersConverterWrapper.converterPrepare,#converterPrepare.logAggregators.size() == 1 ? "assertion" : #assert_fail[444],#converterPrepare.logAggregators[0] == #logAggregatorOutQueue ? "assertion" : #assert_fail[444],#preparedRecordOutQueue=#converterPrepare.preparedRecordQueue,#preparedRecordOutQueueSize=#preparedRecordOutQueue.size(),"------","preparedRecordOutQueue -> com.taobao.drc.logminer.LogRecordConverterCore#run -> logRecordConverterOutQueue",#converterCore = #generator.serializersConverterWrapper.converterCore,#converterCore.inQueue == #preparedRecordOutQueue ? "assertion" : #assert_fail[444],#logRecordConverterOutQueue = #converterCore.outQueue,#logRecordConverterOutQueueSize=#logRecordConverterOutQueue["in"].size(),"------","logRecordConverterOutQueue -> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordDispatcher#run -> dispatchOutIds, dispatchOutRecords, dispatchOutQueue,","dispatchOutQueue -> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSelector#run -> dispatchOutRecords",#logRecordConverterOutQueue == #generator.queue ? "assertion" : #assert_fail[444],#dispatchOutIds = #generator.queue.ids,#dispatchOutRecords = #generator.queue.records,#dispatchOutQueue = #generator.queue.queue,#dispatchOutIdsSize = #dispatchOutIds.size(),#dispatchOutRecordsSize = #dispatchOutRecords.size(),#dispatchOutQueueSize = #dispatchOutQueue.size(),{#dispatchOutIdsSize, #dispatchOutRecordsSize, #dispatchOutQueueSize},"------","dispatchOutIds,dispatchOutRecords -> com.taobao.drc.logminer.queue.OracleBackRecordQueue.RecordSorter#run -> LogminerWrapper.scnIncrementRecordQueue",#scnIncrementRecordQueueInSize = #generator.queue.getAfterSelectQueueSize(),"------","result:",{"fetcherOutQueueSize=" + #fetcherOutQueueSize,"extractorOutQueueSize=" + #extractorOutQueueSize,"logAggregatorOutQueueSize=" + #logAggregatorOutQueueSize,"preparedRecordOutQueueSize=" + #preparedRecordOutQueueSize,"logRecordConverterOutQueueSize=" + #logRecordConverterOutQueueSize,"dispatchOutIdsSize=" + #dispatchOutIdsSize,"dispatchOutRecordsSize=" + #dispatchOutRecordsSize,"dispatchOutQueueSize=" + #dispatchOutQueueSize,"scnIncrementRecordQueueInSize=" + #scnIncrementRecordQueueInSize}' | plaintext
vmtool --action getInstances --className com.taobao.drc.logminer.connect.LogMinerConnectorTask --express '"------","LogminerWrapper.scnIncrementRecordQueue -> com.taobao.drc.logminer.connect.LogMinerConnectorTask.TaskRecordQueue#run ->sourceRecordQueue",#sourceRecordQueueSize = instances[0].taskRecordQueue.records.size(),"------","sourceRecordQueue -> com.taobao.drc.logminer.connect.LogMinerConnectorTask.poll() -> kafkaorg.apache.kafka.connect.runtime.WorkerSourceTask.execute)","kafa configuration in /u01/ds/store/store7100/conf/deliver2store.conf","------",{"sourceRecordQueueSize="+#sourceRecordQueueSize}' | plaintext
bash-4.2$ cat 2.as
 vmtool --action getInstances --className com.oceanbase.connector.framework.CoordinatorContext --express ' #coodinator=instances[0], #sourceConfig=#coodinator.sourceContext.config.sectionKeyValueMap["source"]["clients"], "TODO: check config to match source", "------", "store RPC -> DRCClientImpl.notifyDataMessage -> com.oceanbase.oms.connector.source.store.TransactionAssembler.notify -> com.oceanbase.oms.connector.batch.CommonTransactionAssembler.notify -> sourceOutQueue", #sourceOutQueue=#coodinator.sourceTaskManager.sink, #sourceOutQueueSize=#sourceOutQueue.getAccumulatedBatchSize(), "------", "sourceOutQueue -> bridge thread[0](ETLProcessor) -> bridgeThread0OutQueue", #bridgeThreads=#coodinator.bridgeTaskManager.threads.toArray(), #bridgeThreads.length == 2 ? "assertion" : #assert_fail[444], #bridgeThread0=#bridgeThreads[0], #bridgeThread0Name=#bridgeThread0.threadName, #bridgeThread0.source ==  #sourceOutQueue ? "assertion" : #assert_fail[444], #bridgeThread0OutQueue=#bridgeThread0.sink, #bridgeThread0OutQueueSize=#bridgeThread0OutQueue.getAccumulatedBatchSize(), "------", "bridgeThread0OutQueue -> bridge thread[1](TransactionScheduler) -> bridgeThread1OutQueue", #bridgeThread1=#bridgeThreads[1], #bridgeThread1Name=#bridgeThread1.threadName, #bridgeThread1.source ==  #bridgeThread0OutQueue ? "assertion" : #assert_fail[444], #bridgeThread1OutQueue=#bridgeThread1.sink, #bridgeThread1OutQueueSize=#bridgeThread1OutQueue.getAccumulatedBatchSize(), "------", "bridgeThread1OutQueue -> n * com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask#run -> OBOracleJDBCSink", #coodinator.sinkTaskManager.source == #bridgeThread1OutQueue ?  "assertion" : #assert_fail[444], #sinkTasks=#coodinator.sinkTaskManager.threads, #sinkTasksCount=#sinkTasks.size(), #sinkTasks.toArray()[0].threadName, "TODO: how to get running-status and current sql of DefaultJDBCSink", "------", {"sourceOutQueueSize=" + #sourceOutQueueSize, "bridgeThread0OutQueueSize=" + #bridgeThread0OutQueueSize, "bridgeThread1OutQueueSize=" + #bridgeThread1OutQueueSize, "sinkTasksCount=" + #sinkTasksCount} '
bash-4.2$ cat get_status.sh
java -jar arthas-boot.jar -f 1.as $(ps aux | grep [o]rg.apache.kafka.connect.cli.ConnectDRCDeliver | awk '{print $2}') | grep '    @String'
java -jar arthas-boot.jar -f 2.as --telnet-port 9998 --http-port -1 $(ps aux | grep [c]om.oceanbase.oms.connector.jdbc.coordinator.Bootstrap | awk '{print $2}') | grep '    @String'
bash-4.2$
``` 

效果: 

![image2023-10-26 20:13:13.png](/assets/01KJBZ3EQK7E6D34480VFBNXRD/image2023-10-26%2020%3A13%3A13.png)

# TODO

  - 如何诊断当前环境的问题? (fetcher前的获取有问题? )
  - 获取sink的活跃状况, 正在进行的SQL
  - 获取队列线程的正在进行的任务 
  - STRANGE: LogminerWrapper.scnIncrementRecordQueue 是个静态队列. 其他队列都是generator生成的
