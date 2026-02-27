---
title: 20231117 - 在Oracle并发压力中, 测试OMS性能 [1]
confluence_page_id: 2589492
created_at: 2023-11-17T03:28:09+00:00
updated_at: 2023-11-23T07:10:24+00:00
---

# 场景1

使用hammerDB, vu=2, TPCC压力, 默认配置 iterations=10000000

同步很慢, 队列情况: 

```
bash-4.2$ ARTHAS_PATH=../../arthas-boot.jar bash get_queue_status.sh
../../arthas-boot.jar
[
  "1: LogEntryFetcher_OutQueue_Size=0",
  "2: OracleLogExtractor_OutQueue_Size=0",
  "3: LogRecordSerializer_OutQueue_Size=0",
  "4: LogRecordConverterPrepare_OutQueue_Size=0",
  "5: LogRecordConverterCore_OutQueue_Size=0",
  "6: RecordDispatcher_OutQueue_Ids_Size=0",
  "6: RecordDispatcher_OutQueue_Size=0",
  "6/7.1: RecordDispatcher_OutQueue_Records_Size=0",
  "7.2: global: scnIncrementRecordQueue_Size=0"
]
[
  "8: TaskRecordQueue_OutQueue_Size=0"
]
[
  "11: StoreSource_OutQueue_Size=32",
  "12: ETLProcessor_OutQueue_Size=32",
  "13: TransactionScheduler_OutQueue_Size=0",
  "sinkTasksCount=64"
]
``` 

判断阻塞点在于: TransactionScheduler. 

使用arthas trace功能, 拉取消费长于100ms的操作: 

```
[arthas@74056]$ trace -E com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler offer '#cost > 100'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 125 ms, listenerId: 21
`---ts=2023-11-17 13:25:55;thread_name=queue_slot1-()-TransactionScheduler;id=61;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[309.137056ms] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:offer()
        +---[0.00% 0.003595ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:batchImplClass() #352
        +---[0.00% 0.002954ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:batchImplClass() #355
        +---[0.00% 0.003388ms ] com.google.common.base.Preconditions:checkArgument() #355
        +---[0.00% 0.002125ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:batchType() #358
        +---[0.00% 0.005239ms ] com.oceanbase.oms.connector.dispatcher.scheduler.RecordChecker:checkDiff() #358
        +---[99.97% 309.053277ms ] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:submitTransaction() #366
        +---[0.00% 0.004532ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:getTraceInfo() #367
        `---[0.00% 0.004081ms ] com.oceanbase.oms.connector.common.util.TraceUtil:addTrace() #367
``` 

问题集中于submitTransaction过程, 持续下挖后: 

```
[arthas@74056]$ trace -E com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler offer|submitTransaction|submitDmlTransaction '#cost > 100'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 3) cost in 114 ms, listenerId: 23
`---ts=2023-11-17 13:27:45;thread_name=queue_slot1-()-TransactionScheduler;id=61;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[0.260715ms] com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBroker:submit() #192

`---ts=2023-11-17 13:27:45;thread_name=queue_slot1-()-TransactionScheduler;id=61;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922

`---ts=2023-11-17 13:27:45;thread_name=queue_slot1-()-TransactionScheduler;id=61;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    +---[0.004045ms] com.oceanbase.oms.connector.common.batch.RecordBatch:getTraceInfo() #367
    `---[0.003843ms] com.oceanbase.oms.connector.common.util.TraceUtil:addTrace() #367

`---ts=2023-11-17 13:27:45;thread_name=queue_slot1-()-TransactionScheduler;id=61;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[570.068772ms] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:offer()
        +---[0.00% 0.004553ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:batchImplClass() #352
        +---[0.00% 0.002741ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:batchImplClass() #355
        +---[0.00% 0.002763ms ] com.google.common.base.Preconditions:checkArgument() #355
        +---[0.00% 0.002954ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:batchType() #358
        +---[0.00% 0.005038ms ] com.oceanbase.oms.connector.dispatcher.scheduler.RecordChecker:checkDiff() #358
        +---[99.98% 569.983202ms ] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:submitTransaction() #366
        |   `---[100.00% 569.968053ms ] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:submitTransaction()
        |       +---[0.00% 0.004267ms ] com.oceanbase.oms.connector.common.transaction.Transaction:getTraceInfo() #150
        |       +---[0.00% 0.003082ms ] com.oceanbase.oms.connector.common.util.TraceUtil:addTrace() #150
        |       +---[0.00% 0.00392ms ] com.oceanbase.oms.connector.common.transaction.Transaction:batchType() #151
        |       +---[0.00% 0.00821ms ] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:resetRecordBatchCallBack() #152
        |       `---[99.99% 569.917301ms ] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:submitSingleTransaction() #154
        |           `---[100.00% 569.897137ms ] com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler:submitDmlTransaction()
        |               +---[0.00% 0.004114ms ] com.oceanbase.oms.connector.common.transaction.Transaction:records() #182
        |               `---[0.05% 0.296955ms ] com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBroker:submit() #192
        +---[0.00% 0.007124ms ] com.oceanbase.oms.connector.common.batch.RecordBatch:getTraceInfo() #367
        `---[0.00% 0.004923ms ] com.oceanbase.oms.connector.common.util.TraceUtil:addTrace() #367
``` 

消耗均在 submitSingleTransaction 中. 根据代码分析, DefaultTransactionScheduler 维护了一套数据计数, 用于控制内存消耗, 大致逻辑如下: 

  1. 在事务 给到 处理器TransactionScheduler时, 将事务中的数据行数 累加到 数据计数
     1. 当数据计数达到限制时, 进行等待 (也就是本例中的阻塞)
  2. 当事务被Sink消费完成 (已经完全回放到目标数据库中)时, 将事务中的数据行数 从数据计数中 去除

调整队列查看的脚本, 效果: 

```
bash-4.2$ ARTHAS_PATH=../../arthas-boot.jar bash get_queue_status.sh
../../arthas-boot.jar
[
  "1: LogEntryFetcher_OutQueue_Size=0",
  "2: OracleLogExtractor_OutQueue_Size=0",
  "3: LogRecordSerializer_OutQueue_Size=0",
  "4: LogRecordConverterPrepare_OutQueue_Size=0",
  "5: LogRecordConverterCore_OutQueue_Size=0",
  "6: RecordDispatcher_OutQueue_Ids_Size=0",
  "6: RecordDispatcher_OutQueue_Size=0",
  "6/7.1: RecordDispatcher_OutQueue_Records_Size=0",
  "7.2: global: scnIncrementRecordQueue_Size=0"
]
[
  "8: TaskRecordQueue_OutQueue_Size=0"
]
[
  "11: StoreSource_OutQueue_Size=32",
  "12: ETLProcessor_OutQueue_Size=32",
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=16365/16384",
  "13: TransactionScheduler_OutQueue_Size=0",
  "sinkTasksCount=64"
]
``` 

增加了对队列13的准入阈值 检查

调大阈值后, 该现象消失

# 场景2

在场景1解决后, 发现每秒大概有10个TPCC order回放: 

```
obclient [OBTEST]> select CURRENT_TIMESTAMP from dual; select count(1) from orders;
+-------------------------------------+
| CURRENT_TIMESTAMP                   |
+-------------------------------------+
| 17-NOV-23 02.23.41.331276 PM +08:00 |
+-------------------------------------+
1 row in set (0.001 sec)

+----------+
| COUNT(1) |
+----------+
|    88517 |
+----------+
1 row in set (0.282 sec)

obclient [OBTEST]> select CURRENT_TIMESTAMP from dual; select count(1) from orders;
+-------------------------------------+
| CURRENT_TIMESTAMP                   |
+-------------------------------------+
| 17-NOV-23 02.23.43.757881 PM +08:00 |
+-------------------------------------+
1 row in set (0.001 sec)

+----------+
| COUNT(1) |
+----------+
|    88537 |
+----------+
1 row in set (0.792 sec)
``` 

所有队列均无积压: 

```
bash-4.2$ ARTHAS_PATH=../../arthas-boot.jar bash get_queue_status.sh
[
  "1: LogEntryFetcher_OutQueue_Size=0",
  "2: OracleLogExtractor_OutQueue_Size=0",
  "3: LogRecordSerializer_OutQueue_Size=0",
  "4: LogRecordConverterPrepare_OutQueue_Size=0",
  "5: LogRecordConverterCore_OutQueue_Size=0",
  "6: RecordDispatcher_OutQueue_Ids_Size=0",
  "6: RecordDispatcher_OutQueue_Size=0",
  "6/7.1: RecordDispatcher_OutQueue_Records_Size=0",
  "7.2: global: scnIncrementRecordQueue_Size=0"
]
[
  "8: TaskRecordQueue_OutQueue_Size=0"
]
[
  "11: StoreSource_OutQueue_Size=0",
  "12: ETLProcessor_OutQueue_Size=0",
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=61577/160000",
  "13: TransactionScheduler_OutQueue_Size=0",
  "sinkTasksCount=64"
]
``` 

查看SQL并发程度, 发现并发很低

诊断store的积压: 

```
源端发出 (并接到ack): 
 
[arthas@66584]$ watch com.taobao.drc.deliver.SimpleDRCMessageHub __ackUntilSupposedMessage '{params[0].positionFileID}' -n100000
ts=2023-11-20 14:36:20; [cost=6.09E-4ms] result=[7277705]
method=com.taobao.drc.deliver.SimpleDRCMessageHub.__ackUntilSupposedMessage location=AtExit
 
 
目标端收到: 
 
[arthas@20530]$ watch com.oceanbase.oms.connector.batch.CommonTransactionAssembler notify '{params[0][0].meta.transactionId, params[0][0].meta.transactionSeq}' 'params[0] instanceof java.util.ArrayList && params[0][0].meta.transactionId != null' -x2 -n1024
 
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler.notify location=AtExit
ts=2023-11-20 14:35:43; [cost=0.024548ms] result=@ArrayList[
    @String[1@7139949],
    @Long[13],
]
``` 

查了10w+个事务

目标端从store获取的信息数量: 

```
[arthas@20530]$ watch com.oceanbase.oms.store.client.impl.DRCClientImpl processDataMessage 'params[0].records.size()'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 86 ms, listenerId: 21
method=com.oceanbase.oms.store.client.impl.DRCClientImpl.processDataMessage location=AtExit
ts=2023-11-18 23:41:31; [cost=0.178026ms] result=@Integer[1]
method=com.oceanbase.oms.store.client.impl.DRCClientImpl.processDataMessage location=AtExit
ts=2023-11-18 23:41:31; [cost=0.03304ms] result=@Integer[1]
method=com.oceanbase.oms.store.client.impl.DRCClientImpl.processDataMessage location=AtExit
ts=2023-11-18 23:41:31; [cost=0.01121ms] result=@Integer[1]
method=com.oceanbase.oms.store.client.impl.DRCClientImpl.processDataMessage location=AtExit
...
``` 

每次从store中仅获取一个event, 效率低

检查store DRC client的配置: 

```
[arthas@20530]$ vmtool --action getInstances --className com.oceanbase.oms.store.client.impl.ServerProxy --express 'instances[0].drcConfig' -x 2
@DRCConfig[
    configures=@HashMap[
        @String[subTopic]:@String[ORACLE_np_58m2tfrh5hk0_58m2yeo4cf6o-1-0],
        @String[client.requireCompleteTxn]:@String[false],
        @String[binlogPassword]:@String[RM_oms],
        @String[server.messageType]:@String[binary],
        @String[binlogUsername]:@String[RM_oms],
        @String[client.connectionTimeout]:@String[120],
        @String[version]:@String[2.0.0],
        @String[askSelfUnit]:@String[false],
        @String[useDrcNet]:@String[true],
        @String[client.socketTimeout]:@String[120],
        @String[server.maxRetriedTimes]:@String[1000],
        @String[checkpoint.period]:@String[500],
        @String[manager.host]:@String[http://10.186.16.126:8088],
        @String[timestamp]:@String[1700218988],
    ],
    userDefinedParams=@HashMap[
        @String[password]:@String[RM_oms],
        @String[condition]:@String[OBTEST.DISTRICT|OBTEST.ITEM|OBTEST.WAREHOUSE|OBTEST.ORDERS|OBTEST.CUSTOMER|OBTEST.ORDER_LINE|OBTEST.NEW_ORDER|OBTEST.STOCK|OBTEST.HISTORY|],
        @String[instance]:null,
        @String[dbname]:@String[ORACLE_np_58m2tfrh5hk0_58m2yeo4cf6o-1-0],
        @String[drcMark]:@String[drc.t*x_begin4unit_mark_[0-9]*|*.drc_txn],
        @String[groupname]:@String[RM_oms],
    ],
    persists=@HashSet[isEmpty=true;size=0],
    checkpoint=@Checkpoint[
        recordId=null,
        position=null,
        timestamp=@String[1700321314],
        serverId=null,
        DELIMITER=@String[:],
    ],
    filter=@DataFilterV2[
        CASE_INSENSITIVE_FLAG=@RegularEnumSet[isEmpty=false;size=1],
        branchDB=null,
        filterInfoList=@LinkedList[isEmpty=false;size=9],
        isAllMatch=@Boolean[true],
        storeFilter=@String[OBTEST.DISTRICT|OBTEST.ITEM|OBTEST.WAREHOUSE|OBTEST.ORDERS|OBTEST.CUSTOMER|OBTEST.ORDER_LINE|OBTEST.NEW_ORDER|OBTEST.STOCK|OBTEST.HISTORY|],
        haveValidated=@AtomicBoolean[true],
        requires=@HashMap[isEmpty=false;size=1],
        dbTableColsReflectionMap=@HashMap[isEmpty=true;size=0],
    ],
    blackList=@String[OBTEST.drc_txn*|OBTEST.DRC_TXN*],
    recordsPerBatch=@Integer[0],
    maxRetryTimes=@Integer[1000],
    socketTimeout=@Integer[120],
    connectionTimeout=@Integer[120],
    useBinaryFormat=@Boolean[true],
    txnMark=@Boolean[true],
    requireCompleteTxn=@Boolean[false],
    maxRecordsCached=@Integer[10240],
    maxRecordsBatched=@Integer[1024],
    maxTxnsBatched=@Integer[10240],
    maxTimeoutBatched=@Long[500],
    useDrcNet=@Boolean[true],
    drcMarkWorking=@Boolean[false],
    requiredOtherUnitData=@Boolean[false],
    usePublicIp=@Boolean[false],
    useCaseSensitive=@Boolean[false],
    trimLongType=@Boolean[false],
    useCheckCRC=@Boolean[false],
    useIndexIterator=@Boolean[false],
    needUKRecord=@Boolean[false],
    DRC_VERSION=@String[version],
    DRC_MANAGERHOST=@String[manager.host],
    HTTPS_USE=@String[client.https],
    DRC_BINLOGLOGNAME=@String[DRCClient.Binlog],
    DRC_CHECKPOINT_POLLPERIOD=@String[checkpoint.period],
    SERVER_MAX_RETRIES=@String[server.maxRetriedTimes],
    SERVER_MESSAGE_TYPE=@String[server.messageType],
    CLIENT_SO_TIMEOUT=@String[client.socketTimeout],
    CLIENT_CONN_TIMEOUT=@String[client.connectionTimeout],
    CLIENT_REQ_COMP_TXN=@String[client.requireCompleteTxn],
    CLIENT_MAX_RECS_CACHE=@String[client.maxNumOfRecordsCached],
    CLIENT_MAX_RECS_BATCH=@String[client.maxNumOfRecordsPerMessage],
    CLIENT_MAX_TXNS_BATCH=@String[client.maxNumOfTxnsPerMessage],
    CLIENT_MAX_TIMEOUT_BATCH=@String[client.maxTimeoutPerMessage],
    CLIENT_USE_INDEX_ITERATOR=@String[enableIndexIter],
    USER_FILTERCONDITIONS=@String[condition],
    USER_DBNAME=@String[dbname],
    USER_GROUPNAME=@String[groupname],
    USER_IDENTIFICATION=@String[password],
    USER_GROUP=@String[group],
    USER_SUBGROUP=@String[subgroup],
    USER_MYSQL=@String[instance],
    DRC_MARK=@String[drcMark],
    NEED_UK_RECORD=@String[needUKRecord],
    BLACK_REGION_NO=@String[black_region_no],
    DEFAULT_DRC_MARK=@String[drc.t*x_begin4unit_mark_[0-9]*|*.drc_txn],
    USE_DRC_NET=@String[useDrcNet],
    IPMAPS=@String[ipmaps],
    CLIENT_VERSION=@String[client.version],
    CLIENT_VERSION_ID=@String[58_SP],
    REQUIRE_OTHER_UNIT_DATA=@String[askOtherUnit],
    DATA_REQUIRED_OPTION=@String[client.data_acquire_option],
    SELF_UNIT=@String[self_unit],
    OTHER_UNIT=@String[other_unit],
    USER_FILTERSTRICT=@String[strict],
    USER_FILTERWHERE=@String[where],
    POSITION_INFO=@String[Global_position_info:],
    ipportMaps=@HashMap[isEmpty=true;size=0],
    regionId=@String[0],
    blackRegionNo=null,
]
[arthas@20530]$
``` 

未看出异常

检查目标端从DRC读取数据的调用: 

```
watch alibaba.drcnet.connection.Connection readData '{@java.lang.System@nanoTime()}' -b -s
 
 
method=alibaba.drcnet.connection.SingleDecomressConnection.readData location=AtEnter
ts=2023-11-20 14:47:25; [cost=6.87E-4ms] result=@ArrayList[
    @Long[617404627486800],
]
method=alibaba.drcnet.connection.SingleDecomressConnection.readData location=AtExit
ts=2023-11-20 14:47:25; [cost=6.17404627515996E8ms] result=@ArrayList[
    @Long[617404627542965],
]
method=alibaba.drcnet.connection.SingleDecomressConnection.readData location=AtEnter
ts=2023-11-20 14:47:25; [cost=7.15E-4ms] result=@ArrayList[
    @Long[617404627619244],
]
method=alibaba.drcnet.connection.SingleDecomressConnection.readData location=AtExit
ts=2023-11-20 14:47:25; [cost=6.17404627648233E8ms] result=@ArrayList[
    @Long[617404627674854],
]
method=alibaba.drcnet.connection.SingleDecomressConnection.readData location=AtEnter
ts=2023-11-20 14:47:25; [cost=7.15E-4ms] result=@ArrayList[
    @Long[617404627750550],
]
method=alibaba.drcnet.connection.SingleDecomressConnection.readData location=AtExit
ts=2023-11-20 14:47:25; [cost=6.17404627778825E8ms] result=@ArrayList[
    @Long[617404627806616],
]
``` 

发现: 

  - 单次调用, 大约需要 56000ns
  - 调用间, 大约需要 70000ns
  - 获取一段时间, 找到异常差异点: 均在两轮调用之间
    - ![image2023-11-20 15:3:41.png](/assets/01KJBZ449CMKM8CQ045S94KF5E/image2023-11-20%2015%3A3%3A41.png)

逐层诊断两次从DRC抽取数据之间的操作消耗: 

```
trace com.oceanbase.oms.connector.batch.CommonTransactionAssembler notify '#cost > 100'

`---ts=2023-11-20 15:20:22;thread_name=DRC-Client-Thread;id=63;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[136.695243ms] com.oceanbase.oms.connector.source.store.TransactionAssembler:notify()
        +---[0.01% 0.013787ms ] com.oceanbase.oms.connector.source.store.TransactionAssembler:convertDRCNetBinaryRecordToRecord() #95
        `---[99.96% 136.640074ms ] com.oceanbase.oms.connector.source.store.TransactionAssembler:notify() #96
            `---[99.97% 136.601859ms ] com.oceanbase.oms.connector.batch.CommonTransactionAssembler:notify()
                +---[0.01% 0.008073ms ] com.oceanbase.oms.record.Record:meta() #103
                +---[0.01% 0.019639ms ] com.oceanbase.oms.connector.common.util.RecordUtils:getRecordTimestampMs() #103
                +---[99.70% 136.186894ms ] com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner:generateTransaction() #105
                +---[0.02% 0.025565ms ] com.oceanbase.oms.record.Record:getRecordType() #106
                +---[0.02% 0.025948ms ] com.oceanbase.oms.connector.common.LazyMeasuredGauge:measure() #107
                +---[0.02% 0.025112ms ] com.oceanbase.oms.record.Record:getRecordType() #109
                +---[0.02% 0.025392ms ] com.oceanbase.oms.record.RecordType:ordinal() #109
                +---[0.02% 0.026114ms ] com.oceanbase.oms.record.Record:dataSize() #119
                `---[0.02% 0.025408ms ] com.oceanbase.oms.connector.common.LazyMeasuredGauge:measure() #119
 
 
trace com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner generateTransaction '#cost > 100'
 
`---ts=2023-11-20 15:21:35;thread_name=DRC-Client-Thread;id=63;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[620.601036ms] com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner:generateTransaction()
        +---[0.00% 0.008154ms ] com.oceanbase.oms.record.Record:meta() #199
        +---[0.00% 0.008202ms ] com.oceanbase.oms.record.Record:getRecordType() #201
        +---[0.00% 0.008185ms ] com.oceanbase.oms.record.RecordType:ordinal() #201
        +---[0.01% 0.047161ms ] com.oceanbase.oms.connector.common.transaction.TransactionCutter:finishOneTransaction() #209
        +---[0.00% 0.014336ms ] com.oceanbase.oms.connector.common.transaction.Transaction:setBatchType() #211
        +---[0.01% 0.031219ms ] com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner:getCheckpoint() #212
        +---[0.00% 0.013929ms ] com.oceanbase.oms.connector.common.transaction.Transaction:setCheckpoint() #212
        `---[99.96% 620.322341ms ] com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner:putTransaction() #213
 
 
trace com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner putTransaction '#cost > 100'
 
`---ts=2023-11-20 15:22:43;thread_name=DRC-Client-Thread;id=63;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[197.597918ms] com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner:putTransaction()
        +---[0.01% 0.013176ms ] com.oceanbase.oms.record.Record:getTimestamp() #288
        +---[0.00% 0.009652ms ] com.oceanbase.oms.connector.common.transaction.Transaction:records() #289
        +---[0.00% 0.009634ms ] com.oceanbase.oms.connector.common.transaction.Transaction:resetBatchStateListener() #297
        +---[0.01% 0.010055ms ] com.oceanbase.oms.connector.common.checkpoint.TransactionCheckpointManager:add() #300
        +---[0.01% 0.009891ms ] com.oceanbase.oms.connector.common.log.MsgUtils:getInstance() #313
        +---[0.00% 0.009082ms ] com.oceanbase.oms.connector.common.transaction.Transaction:records() #313
        +---[0.00% 0.009327ms ] com.oceanbase.oms.record.Record:meta() #313
        +---[0.00% 0.009511ms ] com.oceanbase.oms.record.Meta:getSourceIdentity() #313
        +---[0.41% 0.809762ms ] com.oceanbase.oms.connector.common.log.MsgUtils:printSourceRecords() #313
        +---[0.40% min=0.00916ms,max=0.046856ms,total=0.793148ms,count=39] com.oceanbase.oms.connector.common.model.IOffer:offer() #314
        +---[98.18% min=5.03476ms,max=5.151493ms,total=193.999711ms,count=38] com.oceanbase.oms.connector.common.model.IOffer:tryWaitEmpty() #315
        `---[0.02% 0.037171ms ] com.oceanbase.oms.connector.common.model.IOffer:tryWakeUpWaitFillStat() #321
``` 

查看到下游阻塞

重新获取队列情况: 

```
bash-4.2$ ARTHAS_PATH=../../arthas-boot.jar bash get_queue_status.sh
../../arthas-boot.jar
[
  "1: LogEntryFetcher_OutQueue_Size=0",
  "2: OracleLogExtractor_OutQueue_Size=0",
  "3: LogRecordSerializer_OutQueue_Size=0",
  "4: LogRecordConverterPrepare_OutQueue_Size=0",
  "5: LogRecordConverterCore_OutQueue_Size=0",
  "6: RecordDispatcher_OutQueue_Ids_Size=0",
  "6: RecordDispatcher_OutQueue_Size=0",
  "6/7.1: RecordDispatcher_OutQueue_Records_Size=0",
  "7.2: global: scnIncrementRecordQueue_Size=0"
]
[
  "8: TaskRecordQueue_OutQueue_Size=0"
]
Picked up JAVA_TOOL_OPTIONS:
[
  "11: StoreSource_OutQueue_Size=32",
  "12: ETLProcessor_OutQueue_Size=32",
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=159985/160000",
  "13: TransactionScheduler_OutQueue_Size=0",
  "sinkTasksCount=64"
]
``` 

还是场景1的问题, 修复后再测试

调大后, 看到full gc: 

![image2023-11-20 15:39:29.png](/assets/01KJBZ449CMKM8CQ045S94KF5E/image2023-11-20%2015%3A39%3A29.png)

调整内存后, 消耗8.4g内存: 

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
16203 ds        20   0   51.9g   8.4g  24532 S 129.2  3.3  26:04.49 java
70550 ds        20   0 3915932   1.2g   6096 S   8.6  0.5 411:46.62 store
``` 

队列仍然在13队列的限流阀 上积压: 

```
bash-4.2$ ARTHAS_PATH=../../arthas-boot.jar bash get_queue_status.sh
../../arthas-boot.jar
[
  "1: LogEntryFetcher_OutQueue_Size=0",
  "2: OracleLogExtractor_OutQueue_Size=0",
  "3: LogRecordSerializer_OutQueue_Size=0",
  "4: LogRecordConverterPrepare_OutQueue_Size=0",
  "5: LogRecordConverterCore_OutQueue_Size=0",
  "6: RecordDispatcher_OutQueue_Ids_Size=0",
  "6: RecordDispatcher_OutQueue_Size=0",
  "6/7.1: RecordDispatcher_OutQueue_Records_Size=0",
  "7.2: global: scnIncrementRecordQueue_Size=0"
]
[
  "8: TaskRecordQueue_OutQueue_Size=0"
]
[
  "11: StoreSource_OutQueue_Size=0",
  "12: ETLProcessor_OutQueue_Size=0",
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=925534/1000000",
  "13: TransactionScheduler_OutQueue_Size=0",
  "sinkTasksCount=64"
]
``` 

认为Sink的效率有问题. 经过诊断, 发现ConflictBrokerV1冲突检测的效率问题:

```
获取冲突事务数超过100的冲突检测的冲突事务数 (Hash[唯一键] = List(事务))
watch com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1 buildConflictList 'params[2].get(params[0]).size()' 'params[2].get(params[0]).size()>100'
 
获取冲突事务数超过100的冲突检测 的 唯一键个数: 
 
[arthas@45033]$ watch com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1 buildConflictList 'params[2].size()' 'params[2].get(params[0]).size()>100'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 118 ms, listenerId: 6
ts=2023-11-20 22:56:06; [cost=0.007219ms] result=@Integer[20]
method=com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1.buildConflictList location=AtExit
...
ts=2023-11-20 22:56:06; [cost=0.042648ms] result=@Integer[2]
method=com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1.buildConflictList location=AtExit
...
``` 

冲突集中于 两张表, 一张有2个唯一值, 一张有20个唯一值

判断是: WAREHOUSE和DISTRICT两张表

```
[arthas@45033]$ watch com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1 buildConflictList 'target.conflictMap.get("OBTEST").get("WAREHOUSE").size' -n 1
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 110 ms, listenerId: 9
method=com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1.buildConflictList location=AtExit
ts=2023-11-20 22:57:51; [cost=0.213754ms] result=@Integer[2]
 
 
[arthas@45033]$ watch com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1 buildConflictList 'target.conflictMap.get("OBTEST").get("DISTRICT").size' -n 1
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 112 ms, listenerId: 11
method=com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1.buildConflictList location=AtExit
ts=2023-11-20 22:59:24; [cost=0.234423ms] result=@Integer[20]
``` 

查看变更的记录: 

```
watch com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1 buildConflictList 'params[1].records' 'params[2].get(params[0]).size()>100' -n1
``` 

![image2023-11-20 23:10:56.png](/assets/01KJBZ449CMKM8CQ045S94KF5E/image2023-11-20%2023%3A10%3A56.png)

变更的是WAREHOUSE.W_YTD

对于WAREHOUSE.W_YTD的说明: 

  - ![image2023-11-20 23:16:9.png](/assets/01KJBZ449CMKM8CQ045S94KF5E/image2023-11-20%2023%3A16%3A9.png)
  - 在payment事务中:   

    - ![image2023-11-20 23:17:25.png](/assets/01KJBZ449CMKM8CQ045S94KF5E/image2023-11-20%2023%3A17%3A25.png)

判断, 在此处进行事务序列化时, 确实限制了事务的并发

待测试: ConflictBrokerV2

原理分析: [20231123 - OMS ConflictBrokerV2 解析]

在目标端java元数据中, 增加: 

```
coordinator.hotKeyMerge = true
sink.hotKeyTable = [{schema:"OBTEST", tableName:"WAREHOUSE"},{schema:"OBTEST", tableName:"DISTRICT"}]
``` 

重启目标端后, 性能大幅改善, 在场景2压力下, 仅保有少量延迟. 相关日志: 

```
[2023-11-21 13:02:34.183] [WARN] [sinkTask-9] [db:OBTEST, table:DISTRICT, conflict key:5006233774911588015, deepSize:2187, try to merge]
[2023-11-21 13:02:34.185] [WARN] [sinkTask-19] [db:OBTEST, table:DISTRICT, conflict key:5006234874423216226, deepSize:2114, try to merge]
[2023-11-21 13:02:36.032] [WARN] [sinkTask-23] [db:OBTEST, table:DISTRICT, conflict key:-8454801812545027082, deepSize:2092, try to merge]
[2023-11-21 13:02:37.004] [WARN] [sinkTask-39] [db:OBTEST, table:DISTRICT, conflict key:-8733435790737802314, deepSize:2039, try to merge]
[2023-11-21 13:02:44.290] [WARN] [sinkTask-58] [db:OBTEST, table:DISTRICT, conflict key:-6674305644788954481, deepSize:1978, try to merge]
[2023-11-21 13:02:44.291] [WARN] [sinkTask-6] [db:OBTEST, table:DISTRICT, conflict key:5006229376865075171, deepSize:1920, try to merge]
[2023-11-21 13:02:45.024] [WARN] [sinkTask-36] [db:OBTEST, table:DISTRICT, conflict key:5006239272469729070, deepSize:1953, try to merge]
[2023-11-21 13:02:47.020] [WARN] [sinkTask-49] [db:OBTEST, table:DISTRICT, conflict key:4403456939542729913, deepSize:1812, try to merge]
[2023-11-21 13:02:48.016] [WARN] [sinkTask-2] [db:OBTEST, table:DISTRICT, conflict key:5006232675399959804, deepSize:1888, try to merge]
[2023-11-21 13:02:48.072] [WARN] [sinkTask-23] [db:OBTEST, table:DISTRICT, conflict key:7286334158983348077, deepSize:1863, try to merge]
[2023-11-21 13:03:11.593] [INFO] [DRC-Client-Thread] [notify consume: 0.026267382826870724 ms]
[2023-11-21 13:04:01.020] [WARN] [sinkTask-45] [db:OBTEST, table:WAREHOUSE, conflict key:-5808609649712063748, deepSize:9800, try to merge]
[2023-11-21 13:04:12.294] [INFO] [DRC-Client-Thread] [notify consume: 0.028146961030827186 ms]
``` 

FIXME: 

  1. 需要改进日志, 增加V1冲突检测器中, 热点表/键的定期输出. 
  2. 需要改进日志, 增加V2冲突检测器中, 热点表/键的合并操作的输出, 以及合并后仍然存在的热点表/键

# 场景3

hammerdb, 1 warehouse, 32并发

回放差异 增大很多, 回放比场景2慢, 队列情况: 

```
bash-4.2$ ARTHAS_PATH=../../arthas-boot.jar bash get_queue_status.sh
../../arthas-boot.jar
[
  "1: LogEntryFetcher_OutQueue_Size=0",
  "2: OracleLogExtractor_OutQueue_Size=0",
  "3: LogRecordSerializer_OutQueue_Size=0",
  "4: LogRecordConverterPrepare_OutQueue_Size=0",
  "5: LogRecordConverterCore_OutQueue_Size=0",
  "6: RecordDispatcher_OutQueue_Ids_Size=2917",
  "6: RecordDispatcher_OutQueue_Size=0",
  "6/7.1: RecordDispatcher_OutQueue_Records_Size=2911",
  "7.2: global: scnIncrementRecordQueue_Size=0"
]
[
  "8: TaskRecordQueue_OutQueue_Size=19"
]
Picked up JAVA_TOOL_OPTIONS:
[
  "11: StoreSource_OutQueue_Size=256",
  "12: ETLProcessor_OutQueue_Size=256",
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=999748/1000000",
  "13: TransactionScheduler_OutQueue_Size=2485",
  "sinkTasksCount=64"
]
``` 

回放端火焰图: 

[1.html](/assets/01KJBZ449CMKM8CQ045S94KF5E/1.html)

分析: 

  - 其中 com/oceanbase/jdbc/internal/util/Utils.trimSQLString 消耗25% cpu
  - 其中 com/oceanbase/oms/connector/common/log/MsgUtils.printSinkRecords 消耗 25% CPU

待调优

# 技巧: 查看SQL并发运行程度

```
vmtool --action getInstances --className com.oceanbase.jdbc.OceanBaseStatement --express 'instances.{? #this.connection != null && #this.lock.getOwner() != null }.{{#this.lock.getOwner().getName(), #this.actualSql}}' -x 2 --limit 100000
 
输出样例: 
 
[arthas@20530]$ vmtool --action getInstances --className com.oceanbase.jdbc.OceanBaseStatement --express 'instances.{? #this.connection != null && #this.lock.getOwner() != null }.{{#this.lock.getOwner().getName(), #this.actualSql}}' -x 2 --limit 100
@ArrayList[
    @ArrayList[
        @String[sinkTask-16],
        @String[DELETE FROM "OBTEST"."STOCK" WHERE "OMS_OBJECT_NUMBER" = ? and "OMS_RELATIVE_FNO" = ? and "OMS_BLOCK_NUMBER" = ? and "OMS_ROW_NUMBER" = ?],
    ],
    @ArrayList[
        @String[sinkTask-29],
        @String[INSERT INTO "OBTEST"."ORDERS" ("O_ID","O_W_ID","O_D_ID","O_C_ID","O_CARRIER_ID","O_OL_CNT","O_ALL_LOCAL","O_ENTRY_D","OMS_OBJECT_NUMBER","OMS_RELATIVE_FNO","OMS_BLOCK_NUMBER","OMS_ROW_NUMBER") VALUES (?,?,?,?,?,?,?,?,?,?,?,?)],
    ],
]
[arthas@20530]$ vmtool --action getInstances --className com.oceanbase.jdbc.OceanBaseStatement --express 'instances.{? #this.connection != null && #this.lock.getOwner() != null }.{{#this.lock.getOwner().getName(), #this.actualSql}}' -x 2 --limit 100
^[[A@ArrayList[
    @ArrayList[
        @String[sinkTask-45],
        @String[DELETE FROM "OBTEST"."STOCK" WHERE "OMS_OBJECT_NUMBER" = ? and "OMS_RELATIVE_FNO" = ? and "OMS_BLOCK_NUMBER" = ? and "OMS_ROW_NUMBER" = ?],
    ],
    @ArrayList[
        @String[sinkTask-29],
        @String[INSERT INTO "OBTEST"."ORDERS" ("O_ID","O_W_ID","O_D_ID","O_C_ID","O_CARRIER_ID","O_OL_CNT","O_ALL_LOCAL","O_ENTRY_D","OMS_OBJECT_NUMBER","OMS_RELATIVE_FNO","OMS_BLOCK_NUMBER","OMS_ROW_NUMBER") VALUES (?,?,?,?,?,?,?,?,?,?,?,?)],
    ],
]
``` 

比较便宜的方式, 但不能输出SQL: 

```
vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
``` 

对driver的理解: Connection绑定一个Protocol, Connection生成Statement, 并与Protocol绑定. 之后Connection就不再参与过程, 是Statement (sql) 与 Protocol (通讯) 相互作用

# 技巧: 监听SQL

```
watch com.oceanbase.jdbc.OceanBaseStatement executeInternal '{@java.lang.Thread@currentThread().name, params[0] instanceof String ? params[0] : target.actualSql}' '#cost > 100'
 
样例: 
[arthas@20530]$ watch com.oceanbase.jdbc.OceanBaseStatement executeInternal '{@java.lang.Thread@currentThread().name, params[0] instanceof String ? params[0] : target.actualSql}' '#cost > 100'
method=com.oceanbase.jdbc.JDBC4PreparedStatement.executeInternal location=AtExit
ts=2023-11-18 14:38:50; [cost=4.44088791124261E8ms] result=@ArrayList[
    @String[OBHighWatermarkMonitor_0],
    @String[select /*+query_timeout(5000000)*/ 100 - max(round(memstore_used/memstore_limit*100)) from SYS.GV$OB_MEMSTORE],
]
Press Q or Ctrl+C to abort.
Affect(class count: 4 , method count: 2) cost in 320 ms, listenerId: 14
method=com.oceanbase.jdbc.JDBC4PreparedStatement.executeInternal location=AtExit
ts=2023-11-18 14:38:50; [cost=159.384665ms] result=@ArrayList[
    @String[sinkTask-9],
    @String[DELETE FROM "OBTEST"."WAREHOUSE" WHERE "OMS_OBJECT_NUMBER" = ? and "OMS_RELATIVE_FNO" = ? and "OMS_BLOCK_NUMBER" = ? and "OMS_ROW_NUMBER" = ?],
]
method=com.oceanbase.jdbc.JDBC4PreparedStatement.executeInternal location=AtExit
ts=2023-11-18 14:38:51; [cost=345.009351ms] result=@ArrayList[
    @String[sinkTask-0],
    @String[DELETE FROM "OBTEST"."WAREHOUSE" WHERE "OMS_OBJECT_NUMBER" = ? and "OMS_RELATIVE_FNO" = ? and "OMS_BLOCK_NUMBER" = ? and "OMS_ROW_NUMBER" = ?],
]
``` 

按事务查看: 

```
watch com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink offer 'params[0].size()'
``` 

# 技巧: 查看源端发往store的数据水位

```
watch com.taobao.drc.deliver.SimpleDRCMessageHub __ackUntilSupposedMessage '{params[0].positionFileID}'
``` 

# 技巧: 查看目标端从store接到的数据水位

```
watch com.oceanbase.oms.connector.batch.CommonTransactionAssembler notify '{params[0][0].meta.transactionId, params[0][0].meta.transactionSeq}' 'params[0] instanceof java.util.ArrayList && params[0][0].meta.transactionId != null' -x2
``` 

待确认 源端和目标端的水位 相同
