---
title: 20231121 - 在Oracle并发压力中, 测试OMS性能 [2]
confluence_page_id: 2589648
created_at: 2023-11-21T15:54:27+00:00
updated_at: 2023-11-23T10:52:55+00:00
---

# 场景4

10 warehouse, 10 vu, 仅做buildschema数据初始化, 数据同步慢

队列情况: 

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
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=512167/1000000",
  "13: TransactionScheduler_OutQueue_Size=137090",
  "sinkTasksCount=64"
]
``` 

### 现象1: 活跃连接数少

获取活跃的连接的数量:

```
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'

19
[arthas@24895]$
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
25
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
31
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
25
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
37
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
21
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
6
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
14
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
27
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
16
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
20
``` 

维持在20-40之间

通过OBclient的 show full processlist, 也可以看到连接状况 (连接是一批一批被占用, 批次间空余很长时间)

疑问: TransactionScheduler_OutQueue_Size 显示已经积压, 但数据库连接并不打满, 也就是在Sink的处理过程中, 不单是处理数据库写入, 还处理了其他逻辑, 导致连接使用率不高

通过arthas thread采样, 发现sink的堆栈会停留在realWriteDml中, 使用trace诊断: 

```
[arthas@24895]$ trace com.oceanbase.oms.connector.jdbc.sink.Writer realWriteDML '@java.lang.Thread@currentThread().name == "sinkTask-2"' -n 500
 
`---ts=2023-11-22 00:16:05;thread_name=sinkTask-2;id=22;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[1.586801ms] com.oceanbase.oms.connector.jdbc.sink.Writer:realWriteDML()
        +---[0.19% 0.003026ms ] com.oceanbase.oms.connector.cache.TableSchemaCacheManager:getSchema() #540
        +---[0.49% 0.007719ms ] com.oceanbase.oms.connector.jdbc.sink.statement.StatementBuilder:build() #541
        +---[0.04% 5.63E-4ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getSql() #549
        +---[0.17% 0.002762ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getValueList() #550
        +---[0.04% 5.89E-4ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getWhereConditionIndex() #551
        +---[0.81% 0.012913ms ] com.oceanbase.oms.connector.jdbc.sink.PrepareStatementValueSetter:buildStatement() #550
        +---[0.07% 0.001083ms ] org.slf4j.Logger:isDebugEnabled() #564
        +---[0.07% 0.001129ms ] com.oceanbase.oms.connector.common.log.RecordWithMetric:<init>() #568
        +---[0.04% 7.12E-4ms ] com.oceanbase.oms.connector.common.log.MsgUtils:getInstance() #569
        +---[2.02% 0.031979ms ] com.oceanbase.oms.connector.common.log.MsgUtils:printSinkRecords() #569
        `---[0.06% 9.69E-4ms ] com.oceanbase.oms.connector.common.util.CommonUtil:swallowErrorClose() #579
`---ts=2023-11-22 00:16:05;thread_name=sinkTask-2;id=22;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[0.918266ms] com.oceanbase.oms.connector.jdbc.sink.Writer:realWriteDML()
        +---[0.15% 0.001375ms ] com.oceanbase.oms.connector.cache.TableSchemaCacheManager:getSchema() #540
        +---[0.92% 0.008455ms ] com.oceanbase.oms.connector.jdbc.sink.statement.StatementBuilder:build() #541
        +---[0.47% 0.004288ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getSql() #549
        +---[0.07% 6.4E-4ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getValueList() #550
        +---[0.31% 0.002854ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getWhereConditionIndex() #551
        +---[0.59% 0.005397ms ] com.oceanbase.oms.connector.jdbc.sink.PrepareStatementValueSetter:buildStatement() #550
        +---[0.12% 0.001062ms ] org.slf4j.Logger:isDebugEnabled() #564
        +---[0.12% 0.001091ms ] com.oceanbase.oms.connector.common.log.RecordWithMetric:<init>() #568
        +---[0.09% 8.44E-4ms ] com.oceanbase.oms.connector.common.log.MsgUtils:getInstance() #569
        +---[30.61% 0.281077ms ] com.oceanbase.oms.connector.common.log.MsgUtils:printSinkRecords() #569
        `---[0.15% 0.001414ms ] com.oceanbase.oms.connector.common.util.CommonUtil:swallowErrorClose() #579

`---ts=2023-11-22 00:16:05;thread_name=sinkTask-2;id=22;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    `---[1.268118ms] com.oceanbase.oms.connector.jdbc.sink.Writer:realWriteDML()
        +---[0.12% 0.001469ms ] com.oceanbase.oms.connector.cache.TableSchemaCacheManager:getSchema() #540
        +---[0.61% 0.007691ms ] com.oceanbase.oms.connector.jdbc.sink.statement.StatementBuilder:build() #541
        +---[0.06% 8.08E-4ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getSql() #549
        +---[0.06% 6.98E-4ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getValueList() #550
        +---[0.06% 7.84E-4ms ] com.oceanbase.oms.connector.jdbc.sink.StatementResult:getWhereConditionIndex() #551
        +---[0.73% 0.009213ms ] com.oceanbase.oms.connector.jdbc.sink.PrepareStatementValueSetter:buildStatement() #550
        +---[0.25% 0.003219ms ] org.slf4j.Logger:isDebugEnabled() #564
        +---[0.06% 8.21E-4ms ] com.oceanbase.oms.connector.common.log.RecordWithMetric:<init>() #568
        +---[0.27% 0.003434ms ] com.oceanbase.oms.connector.common.log.MsgUtils:getInstance() #569
        +---[35.39% 0.448816ms ] com.oceanbase.oms.connector.common.log.MsgUtils:printSinkRecords() #569
        `---[0.34% 0.004351ms ] com.oceanbase.oms.connector.common.util.CommonUtil:swallowErrorClose() #579
``` 
    
    
    com.oceanbase.oms.connector.common.log.MsgUtils:printSinkRecords 会占用较多的时间

调整printSinkRecords的行为: 

```
options strict false
vmtool --action getInstances --className com.oceanbase.oms.connector.common.log.MsgUtils --express 'instances[0].printTraceMsg = false'
options strict true
``` 

连接并发数会上升: 

```
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
@Integer[45]
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
@Integer[60]
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
@Integer[41]
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express 'instances.{? #this.lock.isLocked()}.size()'
@Integer[20]
``` 

### 现象2: 出现复制数据不一致, 复制链路故障

目标端有两张表复制不过来

```
Every 2.0s: ./watch_target_db_client.sh                                                             R740-26: Wed Nov 22 13:00:37 2023

+------------+----------+
| 'CUSTOMER' | COUNT(1) |
+------------+----------+
| CUSTOMER   |   300000 |
| DISTRICT   |      100 |
| HISTORY    |   300000 |
| ITEM       |   100000 |
| NEW_ORDER  |        0 |
| ORDERS     |   300000 |
| ORDER_LINE |        0 |
| STOCK      |  1000000 |
| WAREHOUSE  |       10 |
+------------+----------+
``` 

队列状况: 

```
huangyan@R740-26:~$ docker exec -it OMS_20231025_122226 su ds -c "cd /opt/arthas/oms-copilot/get_queue_status/; ARTHAS_PATH=../../arthas-boot.jar bash get_queue_status.sh"
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
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=0/16384",
  "13: TransactionScheduler_OutQueue_Size=0",
  "sinkTasksCount=64"
]
``` 

但复制链路的心跳在正常进行: 

```
[arthas@24895]$ watch com.oceanbase.oms.connector.batch.CommonTransactionAssembler notify 'params[0][0]' 'params[0] instanceof java.util.ArrayList'
Press Q or Ctrl+C to abort.
Affect(class count: 4 , method count: 4) cost in 308 ms, listenerId: 19
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler.notify location=AtExit
ts=2023-11-22 13:03:22; [cost=0.234157ms] result={"checkpoint":"1700629392","conflictInfos":[],"logMessage":{"attributes":{},"byteBuff":{"contiguous":true,"direct":false,"readOnly":false,"readable":true,"writable":false},"checkpoint":"1@19087205","dbType":"ORACLE","dbTypeInXLogMode":false,"encodingStr":"US-ASCII","fieldCount":0,"fileNameOffset":19087205,"fileOffset":1,"firstInLogEvent":false,"id":"0","isConnectionFirstRecord":false,"keyChange":false,"logSeqNum":0,"messageUniqueId":6316501392648810753,"messageUniqueIdStr":"/ORACLE/HEARTBEAT/null/null/19087205/1//1700629392","metaVersion":-1,"opt":"HEARTBEAT","primaryAndUniqueConstraintColumnIndexTuples":[],"primaryKeyIndex":[],"primaryKeys":"","primaryValues":[],"queryBack":false,"rawData":"AwABAM8AAAADAAAAEmgAAAADAwQAAAAAAAAAAAAAAAAAkItdZQAAAACdAAAA/////////////////////////////////////2U/IwEAAAAAAQAAAAAAAACrAAAAsAAAAP////////////////////+1AAAAugAAAP///////////////////////////////////////////////////////////////wsJAAAAVVMtQVNDSUkAGwAAAAAbAAAAABsAAAAAEgwAAAAAAAAAAAAAAAAAAAA=","regionId":"0","safeTimestamp":"1700629392","threadId":"0","timestamp":"1700629392","timestampUsec":"0","traceInfo":"","uniqueColNames":"","version":3},"recordType":"HEARTBEAT","timestamp":1700629392}
``` 

其中 SCN=19087205. 与此同时 源数据库中的SCN: 

```
SQL> SELECT CURRENT_SCN FROM V$DATABASE;

CURRENT_SCN
-----------
   19087313
``` 

判断心跳机制是正常的?? (不确定心跳上的SCN 的来源)

新写入一行数据, 数据在一定时间后开始不同步

```
insert into warehouse (W_ID		,W_YTD		,W_TAX		,W_NAME 	,W_STREET_1,W_STREET_2,W_CITY 	,W_STATE	,W_ZIP) values (200,101010, 0.11, 'aaaa','aaaa','aaaa','aaaa','aa','aaaa');
``` 

目标端的冲突检测列表也没有数据: 

```
[arthas@24895]$ vmtool --action getInstances --className com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV1 --express 'instances[0].conflictMap.size()'
0
``` 

仿佛NEW_ORDER和ORDER_LINE的数据都消失了

重启了源端+store+目标端, 这些数据仍然不同步, 但新数据会同步. 

清理所有数据, 再次写入所有数据, 还是这两张表没有数据.

在源数据库插入相关表的数据: 

```
insert into new_order (no_w_id, no_d_id, no_o_id) values (1,1,1);
``` 

仍然不会复制到目标端

诊断过程链路: 

```
处理器1可以看到该事务 (从Logminer机制流入了处理链)
 
watch com.taobao.drc.logminer.fetcher.LogEntryFetcher putEntry 'params[0]'
 
---
 
method=com.taobao.drc.logminer.fetcher.LogEntryFetcher.putEntry location=AtExit
ts=2023-11-22 14:28:39; [cost=0.056214ms] result=@OracleLogEntry[
    OP_TYPES=@OpType[][isEmpty=false;size=256],
    SEGMENT_TYPES=@SegmentType[][isEmpty=false;size=48],
    DATE_TIME_CACHE=@ConcurrentHashMap[isEmpty=true;size=0],
    ownerId=null,
    tableId=null,
    scn=@Long[19792035],
    timestamp=@String[2023-11-22 06:28:24],
    xid=@String[1D001800F7120000],
    originXid=@String[1D001800F7120000],
    rowid=@String[AAASMqAAAAAAAAAAAA],
    opType=@OpType[START],
    segmentType=@SegmentType[UNKNOWN],
    segmentName=null,
    redo=@String[set transaction read write;],
    csf=@Short[0],
    threadId=@Integer[1],
    rbaLogFileSeq=@Integer[2234],
    rbaBlockId=@Long[1],
    rbaByteOffset=@Integer[16],
    rollback=@Integer[0],
    info=null,
    rownum=@Long[1285],
    rsId=@String[0x0008ba.000005dd.0010],
]
method=com.taobao.drc.logminer.fetcher.LogEntryFetcher.putEntry location=AtExit
ts=2023-11-22 14:28:39; [cost=0.012874ms] result=@OracleLogEntry[
    OP_TYPES=@OpType[][isEmpty=false;size=256],
    SEGMENT_TYPES=@SegmentType[][isEmpty=false;size=48],
    DATE_TIME_CACHE=@ConcurrentHashMap[isEmpty=true;size=0],
    ownerId=@String[UNKNOWN],
    tableId=@String[OBJ# 73642],
    scn=@Long[19792035],
    timestamp=@String[2023-11-22 06:28:24],
    xid=@String[1D001800F7120000],
    originXid=@String[1D001800F7120000],
    rowid=@String[AAASMqAAAAAAAAAAAA],
    opType=@OpType[INSERT],
    segmentType=@SegmentType[UNKNOWN],
    segmentName=@String[OBJ# 73642],
    redo=@String[insert into "UNKNOWN"."OBJ# 73642"("COL 1","COL 2","COL 3") values (HEXTORAW('c102'),HEXTORAW('c102'),HEXTORAW('c104'));],
    csf=@Short[0],
    threadId=@Integer[1],
    rbaLogFileSeq=@Integer[2234],
    rbaBlockId=@Long[1],
    rbaByteOffset=@Integer[16],
    rollback=@Integer[0],
    info=@String[Dictionary Mismatch],
    rownum=@Long[1286],
    rsId=@String[0x0008ba.000005dd.0010],
]
method=com.taobao.drc.logminer.fetcher.LogEntryFetcher.putEntry location=AtExit
ts=2023-11-22 14:28:39; [cost=0.016498ms] result=@OracleLogEntry[
    OP_TYPES=@OpType[][isEmpty=false;size=256],
    SEGMENT_TYPES=@SegmentType[][isEmpty=false;size=48],
    DATE_TIME_CACHE=@ConcurrentHashMap[isEmpty=true;size=0],
    ownerId=null,
    tableId=null,
    scn=@Long[19792040],
    timestamp=@String[2023-11-22 06:28:24],
    xid=@String[1D001800F7120000],
    originXid=@String[1D001800F7120000],
    rowid=@String[AAAAAAAAAAAAAAAAAA],
    opType=@OpType[COMMIT],
    segmentType=@SegmentType[UNKNOWN],
    segmentName=null,
    redo=@String[commit;],
    csf=@Short[0],
    threadId=@Integer[1],
    rbaLogFileSeq=@Integer[2234],
    rbaBlockId=@Long[1],
    rbaByteOffset=@Integer[16],
    rollback=@Integer[0],
    info=null,
    rownum=@Long[1287],
    rsId=@String[0x0008ba.000005de.0010],
]
```
```
watch com.taobao.drc.logminer.connect.LogMinerConnectorTask poll 'returnObj.{#this.toString()}' 'returnObj.size() > 0'
 
从处理器9, 无法观测到 (没有从源java中流出)
``` 

从处理器3出口观察: 

```
正常事务 (insert into warehouse (W_ID		,W_YTD		,W_TAX		,W_NAME 	,W_STREET_1,W_STREET_2,W_CITY 	,W_STATE	,W_ZIP) values (206,101010, 0.11, 'aaaa','aaaa','aaaa','aaaa','aa','aaaa');):
INSERT的值会出现在Record中: 
 
method=com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator.dealCommittedTransaction location=AtExit
ts=2023-11-22 14:39:24; [cost=0.04714ms] result=@OracleLogTransaction[
...
    bigListRecords=@BigList[
        @OracleLogRecord[{id: 4253159
scn: 19793576
rowid: null
timestamp: 1700635148000
type: START
table: null
transaction: 1.0x0008ba.0000062c.00bc
fileid: 2234
blockid: 1
rollback: 0
rsid:0x0008ba.0000062a.0154
threadid:1
}
],
        @OracleLogRecord[{id: 4253160
scn: 19793576
rowid: AAASMyAAFAAAEgoAAA
timestamp: 1700635148000
type: INSERT
table: 73639
transaction: 1.0x0008ba.0000062c.00bc
fileid: 2234
blockid: 1
rollback: 0
rsid:0x0008ba.0000062a.0154
threadid:1
field name(new): 1
field value(new): c20307
field name(new): 2
field value(new): c30b0b0b
field name(new): 3
field value(new): c00c
field name(new): 4
field value(new): 61616161
field name(new): 5
field value(new): 61616161
field name(new): 6
field value(new): 61616161
field name(new): 7
field value(new): 61616161
field name(new): 8
field value(new): 6161
field name(new): 9
field value(new): 616161612020202020
}
],
        @OracleLogRecord[{id: 4253161
scn: 19793576
rowid: null
timestamp: 1700635148000
type: COMMIT
table: null
transaction: 1.0x0008ba.0000062c.00bc
fileid: 2234
blockid: 1
rollback: 0
rsid:0x0008ba.0000062c.00bc
threadid:1
}
],
    ],
...
]

 
 
异常事务 (insert into new_order (no_w_id, no_d_id, no_o_id) values (1,1,6);)
数据记录消失: 
 
method=com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator.dealCommittedTransaction location=AtExit
ts=2023-11-22 14:41:45; [cost=0.027735ms] result=@OracleLogTransaction[
...
    bigListRecords=@BigList[
        @OracleLogRecord[{id: 4253162
scn: 19793928
rowid: null
timestamp: 1700635298000
type: START
table: null
transaction: 1.0x0008ba.00000630.0010
fileid: 2234
blockid: 1
rollback: 0
rsid:0x0008ba.0000062f.0010
threadid:1
}
],
        @OracleLogRecord[{id: 4253163
scn: 19793928
rowid: null
timestamp: 1700635298000
type: COMMIT
table: null
transaction: 1.0x0008ba.00000630.0010
fileid: 2234
blockid: 1
rollback: 0
rsid:0x0008ba.00000630.0010
threadid:1
}
],
...
]
method=com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator.dealCommittedTransaction location=AtExit
``` 

从处理器3入口观察: 

数据并未进入处理器3

```
[arthas@44893]$ watch com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator put 'params[0]'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 80 ms, listenerId: 8
method=com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator.put location=AtExit
ts=2023-11-22 14:45:26; [cost=50.71352ms] result=@OracleLogRecord[
    TYPES=@OperationType[][isEmpty=false;size=17],
    id=@Long[4280220],
    rowid=null,
    table=null,
    owner=null,
    type=@OperationType[START],
    timestamp=@Long[1700635508000],
    scn=@Long[19794428],
    transaction=@String[1D00110070130000],
    originXid=@String[1D00110070130000],
    fileid=@Integer[2234],
    blockid=@Long[1],
    bytesid=@Integer[16],
    rollback=@Integer[0],
    rsid=@String[0x0008ba.00000647.0010],
    threadid=@Integer[1],
    oldColumns=null,
    newColumns=null,
    heapSize=@Long[-1],
    FIX_HEAPSIE=@Integer[100],
]
method=com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator.put location=AtExit
ts=2023-11-22 14:45:26; [cost=0.992559ms] result=@OracleLogRecord[
    TYPES=@OperationType[][isEmpty=false;size=17],
    id=@Long[4253173],
    rowid=null,
    table=null,
    owner=null,
    type=@OperationType[COMMIT],
    timestamp=@Long[1700635509000],
    scn=@Long[19794433],
    transaction=@String[1.0x0008ba.00000648.0010],
    originXid=@String[1D00110070130000],
    fileid=@Integer[2234],
    blockid=@Long[1],
    bytesid=@Integer[16],
    rollback=@Integer[0],
    rsid=@String[0x0008ba.00000648.0010],
    threadid=@Integer[1],
    oldColumns=null,
    newColumns=null,
    heapSize=@Long[-1],
    FIX_HEAPSIE=@Integer[100],
]
``` 

观察处理器2的入口: 可以看到数据

```
[arthas@44893]$ watch com.taobao.drc.logminer.queue.OracleLogEntryQueue drainTo 'params[0]' -s
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 75 ms, listenerId: 9
method=com.taobao.drc.logminer.queue.OracleLogEntryQueue.drainTo location=AtExit
ts=2023-11-22 15:10:27; [cost=0.108352ms] result=@ArrayList[
    @OracleLogEntry[OracleLogEntry{rownum=6620, ownerId='null', tableId='null', scn=19798281, timestamp='2023-11-22 07:10:11', xid='1D0020005B130000', rowid='AAASMqAAAAAAAAAAAA', opType=START, segmentType=UNKNOWN, segmentName='null', redo='set transaction read write;', csf=0, threadId=1, rbaLogFileSeq=2234, rbaBlockId=1, rbaByteOffset=148, rollback=0, info=null, rsId='0x0008ba.00001143.0094'}],
    @OracleLogEntry[OracleLogEntry{rownum=6621, ownerId='UNKNOWN', tableId='OBJ# 73642', scn=19798281, timestamp='2023-11-22 07:10:11', xid='1D0020005B130000', rowid='AAASMqAAAAAAAAAAAA', opType=INSERT, segmentType=UNKNOWN, segmentName='OBJ# 73642', redo='insert into "UNKNOWN"."OBJ# 73642"("COL 1","COL 2","COL 3") values (HEXTORAW('c102'),HEXTORAW('c102'),HEXTORAW('c109'));', csf=0, threadId=1, rbaLogFileSeq=2234, rbaBlockId=1, rbaByteOffset=148, rollback=0, info=Dictionary Mismatch, rsId='0x0008ba.00001143.0094'}],
    @OracleLogEntry[OracleLogEntry{rownum=6622, ownerId='null', tableId='null', scn=19798289, timestamp='2023-11-22 07:10:12', xid='1D0020005B130000', rowid='AAAAAAAAAAAAAAAAAA', opType=COMMIT, segmentType=UNKNOWN, segmentName='null', redo='commit;', csf=0, threadId=1, rbaLogFileSeq=2234, rbaBlockId=1, rbaByteOffset=16, rollback=0, info=null, rsId='0x0008ba.00001145.0010'}],
``` 

观察处理器2中的分析器入口: 可以看到数据

```
method=com.taobao.drc.logminer.OracleLogExtractor$AnalyserCallable.<init> location=AtExit
ts=2023-11-22 15:15:32; [cost=0.087472ms] result=@ArrayList[
    @OracleLogEntry[OracleLogEntry{rownum=6643, ownerId='null', tableId='null', scn=19799033, timestamp='2023-11-22 07:15:21', xid='18001500E9060000', rowid='AAASMqAAAAAAAAAAAA', opType=START, segmentType=UNKNOWN, segmentName='null', redo='set transaction read write;', csf=0, threadId=1, rbaLogFileSeq=2234, rbaBlockId=1, rbaByteOffset=16, rollback=0, info=null, rsId='0x0008ba.00001163.0010'}],
]
method=com.taobao.drc.logminer.OracleLogExtractor$AnalyserCallable.<init> location=AtExit
ts=2023-11-22 15:15:32; [cost=0.005352ms] result=@ArrayList[
    @OracleLogEntry[OracleLogEntry{rownum=6644, ownerId='UNKNOWN', tableId='OBJ# 73642', scn=19799033, timestamp='2023-11-22 07:15:21', xid='18001500E9060000', rowid='AAASMqAAAAAAAAAAAA', opType=INSERT, segmentType=UNKNOWN, segmentName='OBJ# 73642', redo='insert into "UNKNOWN"."OBJ# 73642"("COL 1","COL 2","COL 3") values (HEXTORAW('c102'),HEXTORAW('c102'),HEXTORAW('c10b'));', csf=0, threadId=1, rbaLogFileSeq=2234, rbaBlockId=1, rbaByteOffset=16, rollback=0, info=Dictionary Mismatch, rsId='0x0008ba.00001163.0010'}],
    @OracleLogEntry[OracleLogEntry{rownum=6645, ownerId='null', tableId='null', scn=19799040, timestamp='2023-11-22 07:15:22', xid='18001500E9060000', rowid='AAAAAAAAAAAAAAAAAA', opType=COMMIT, segmentType=UNKNOWN, segmentName='null', redo='commit;', csf=0, threadId=1, rbaLogFileSeq=2234, rbaBlockId=1, rbaByteOffset=16, rollback=0, info=null, rsId='0x0008ba.00001164.0010'}],
]
``` 

观察处理器2中的分析器出口: 数据消失: 

```
method=com.taobao.drc.logminer.OracleLogExtractor$AnalyserCallable.call location=AtExit
ts=2023-11-22 15:17:33; [cost=0.411503ms] result=@LinkedList[
    @OracleLogRecord[
        TYPES=@OperationType[][isEmpty=false;size=17],
        id=@Long[-1],
        rowid=null,
        table=null,
        owner=null,
        type=@OperationType[START],
        timestamp=@Long[1700637430000],
        scn=@Long[19799290],
        transaction=@String[1D000D007F130000],
        originXid=@String[1D000D007F130000],
        fileid=@Integer[2234],
        blockid=@Long[1],
        bytesid=@Integer[16],
        rollback=@Integer[0],
        rsid=@String[0x0008ba.00001167.0010],
        threadid=@Integer[1],
        oldColumns=null,
        newColumns=null,
        heapSize=@Long[-1],
        FIX_HEAPSIE=@Integer[100],
    ],
]
method=com.taobao.drc.logminer.OracleLogExtractor$AnalyserCallable.call location=AtExit
ts=2023-11-22 15:17:33; [cost=5.038756ms] result=@LinkedList[
    @OracleLogRecord[
        TYPES=@OperationType[][isEmpty=false;size=17],
        id=@Long[-1],
        rowid=null,
        table=null,
        owner=null,
        type=@OperationType[START],
        timestamp=@Long[1700637430000],
        scn=@Long[19799290],
        transaction=@String[1D000D007F130000],
        originXid=@String[1D000D007F130000],
        fileid=@Integer[2234],
        blockid=@Long[1],
        bytesid=@Integer[16],
        rollback=@Integer[0],
        rsid=@String[0x0008ba.00001167.0010],
        threadid=@Integer[1],
        oldColumns=null,
        newColumns=null,
        heapSize=@Long[-1],
        FIX_HEAPSIE=@Integer[100],
    ],
]
method=com.taobao.drc.logminer.OracleLogExtractor$AnalyserCallable.call location=AtExit
ts=2023-11-22 15:17:33; [cost=0.142264ms] result=@LinkedList[
    @OracleLogRecord[
        TYPES=@OperationType[][isEmpty=false;size=17],
        id=@Long[-1],
        rowid=null,
        table=null,
        owner=null,
        type=@OperationType[COMMIT],
        timestamp=@Long[1700637430000],
        scn=@Long[19799293],
        transaction=@String[1D000D007F130000],
        originXid=@String[1D000D007F130000],
        fileid=@Integer[2234],
        blockid=@Long[1],
        bytesid=@Integer[64],
        rollback=@Integer[0],
        rsid=@String[0x0008ba.00001168.0040],
        threadid=@Integer[1],
        oldColumns=null,
        newColumns=null,
        heapSize=@Long[-1],
        FIX_HEAPSIE=@Integer[100],
    ],
]
method=com.taobao.drc.logminer.OracleLogExtractor$AnalyserCallable.call location=AtExit
ts=2023-11-22 15:17:33; [cost=1.560295ms] result=@LinkedList[
    @OracleLogRecord[
        TYPES=@OperationType[][isEmpty=false;size=17],
        id=@Long[-1],
        rowid=null,
        table=null,
        owner=null,
        type=@OperationType[COMMIT],
        timestamp=@Long[1700637430000],
        scn=@Long[19799293],
        transaction=@String[1D000D007F130000],
        originXid=@String[1D000D007F130000],
        fileid=@Integer[2234],
        blockid=@Long[1],
        bytesid=@Integer[64],
        rollback=@Integer[0],
        rsid=@String[0x0008ba.00001168.0040],
        threadid=@Integer[1],
        oldColumns=null,
        newColumns=null,
        heapSize=@Long[-1],
        FIX_HEAPSIE=@Integer[100],
    ],
]

``` 

发现表被过滤掉了: 

```
[arthas@44893]$ watch com.taobao.drc.logminer.log.OracleLogEntryAnalyzer needFilterThisEntry '{params[0], returnObj}'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 74 ms, listenerId: 18
method=com.taobao.drc.logminer.log.OracleLogEntryAnalyzer.needFilterThisEntry location=AtExit
ts=2023-11-22 15:28:03; [cost=0.077604ms] result=@ArrayList[
    @OracleLogEntry[OracleLogEntry{rownum=6692, ownerId='UNKNOWN', tableId='OBJ# 73642', scn=19800793, timestamp='2023-11-22 07:27:44', xid='1D0006002A130000', rowid='AAASMqAAAAAAAAAAAA', opType=INSERT, segmentType=UNKNOWN, segmentName='OBJ# 73642', redo='insert into "UNKNOWN"."OBJ# 73642"("COL 1","COL 2","COL 3") values (HEXTORAW('c102'),HEXTORAW('c102'),HEXTORAW('c10f'));', csf=0, threadId=1, rbaLogFileSeq=2234, rbaBlockId=1, rbaByteOffset=16, rollback=0, info=Dictionary Mismatch, rsId='0x0008ba.0000119c.0010'}],
    @Boolean[true],
]
``` 

被过滤的逻辑, 是因为Object ID无法查询到: 与实际表对不上

```
SQL> select owner, object_id, object_name from all_objects where object_id = 73642;

OWNER				OBJECT_ID OBJECT_NAME
------------------------------ ---------- ------------------------------
OBTEST				    73642 INORD
 
 
OMS中实际用到的查询语句:
 
SQL> select owner, object_name from all_objects where object_id = 73642 and OBJECT_TYPE like 'TABLE%' and TEMPORARY = 'N' and OBJECT_NAME not like 'BIN$%' and (OWNER, OBJECT_NAME) not in (SELECT OWNER, MVIEW_NAME FROM DBA_MVIEWS UNION ALL SELECT LOG_OWNER, LOG_TABLE FROM DBA_MVIEW_LOGS);

no rows selected

``` 

查看OBTEST的表: 

```
SQL> select owner, object_id, object_name from all_objects where OBJECT_TYPE like 'TABLE%' and TEMPORARY = 'N' and OBJECT_NAME not like 'BIN$%' and (OWNER, OBJECT_NAME) not in (SELECT OWNER, MVIEW_NAME FROM DBA_MVIEWS UNION ALL SELECT LOG_OWNER, LOG_TABLE FROM DBA_MVIEW_LOGS) and owner = 'OBTEST';

OWNER				OBJECT_ID OBJECT_NAME
------------------------------ ---------- ------------------------------
OBTEST				    73635 CUSTOMER
OBTEST				    73643 ORDERS
OBTEST				    73641 NEW_ORDER
OBTEST				    73637 HISTORY
OBTEST				    73639 WAREHOUSE
OBTEST				    73636 DISTRICT
OBTEST				    73644 ORDER_LINE
OBTEST				    73640 STOCK
OBTEST				    73638 ITEM

9 rows selected.
``` 

找到73642的定义: 

```
SQL> select * from all_objects where object_id = 73642
  2  ;

OWNER			       OBJECT_NAME
------------------------------ ------------------------------
SUBOBJECT_NAME			OBJECT_ID DATA_OBJECT_ID OBJECT_TYPE
------------------------------ ---------- -------------- -------------------
CREATED   LAST_DDL_ TIMESTAMP		STATUS	T G S  NAMESPACE
--------- --------- ------------------- ------- - - - ----------
EDITION_NAME
------------------------------
OBTEST			       INORD
				    73642	   74538 INDEX
16-NOV-23 22-NOV-23 2023-11-16:10:15:28 VALID	N N N	       4
``` 

是一个索引, 定义是: 

```
SET LONG 10000
SET LONGCHUNKSIZE 1000
SET LINESIZE 1000
SET PAGESIZE 0
SET TRIMOUT ON
SET TRIMSPOOL ON
SELECT
  dbms_metadata.get_ddl('INDEX', 'INORD', 'OBTEST') AS index_definition
FROM
  dual;

 
---
 
  CREATE UNIQUE INDEX "OBTEST"."INORD" ON "OBTEST"."NEW_ORDER" ("NO_W_ID", "NO_D_ID", "NO_O_ID")
  PCTFREE 10 INITRANS 4 MAXTRANS 255 COMPUTE STATISTICS
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1 BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "OBTEST"
``` 

也就是说: NEW_ORDER是一个全索引表, 数据是计入索引中, 而不是表中, 导致根据OBJECT_ID找不到对应的表

再次测试,ORDER_LINE不是全索引表, 需要调查其数据丢失的原因: 

```
INSERT INTO order_line (ol_o_id, ol_d_id, ol_w_id, ol_number, ol_i_id, ol_supply_w_id, ol_quantity, ol_amount, ol_dist_info, ol_delivery_d) values (999, 999, 999, 1, 999, 999, 1, 0, 'a', NULL);
commit;
``` 

使用arthas查看: 表ID是73645, 确实是被跳过了

```
[arthas@44893]$ watch com.taobao.drc.logminer.log.OracleLogEntryAnalyzer needFilterThisEntry '{params[0], returnObj}'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 103 ms, listenerId: 19
method=com.taobao.drc.logminer.log.OracleLogEntryAnalyzer.needFilterThisEntry location=AtExit
ts=2023-11-23 00:19:11; [cost=0.177422ms] result=@ArrayList[
    @OracleLogEntry[OracleLogEntry{rownum=30766, ownerId='UNKNOWN', tableId='OBJ# 383', scn=20032575, timestamp='2023-11-22 16:18:47', xid='1B000F002E0B0000', rowid='AAAAAAAAAAAAAAAAAA', opType=INSERT, segmentType=UNKNOWN, segmentName='OBJ# 383', redo='insert into "UNKNOWN"."OBJ# 383"("COL 1","COL 2","COL 3","COL 4","COL 5","COL 6","COL 7","COL 8","COL 9","COL 10","COL 11","COL 12","COL 13","COL 14","COL 15","COL 16","COL 17","COL 18","COL 19","COL 20","COL 21","COL 22","COL 23","COL 24","COL 25","COL 26","COL 27","COL 28","COL 29","COL 30","COL 31","COL 32","COL 33","COL 34","COL 35","COL 36","COL 37","COL 38","COL 39","COL 40","COL 41","COL 42") values (HEXTORAW('c3075553'),HEXTORAW('c102'),HEXTORAW('c102'),NULL,HEXTORAW('4f4254455354'),HEXTORAW('643333653861323734356665'),HEXTORAW('7074732f3234'),HEXTORAW('c202'),HEXTORAW('80'),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,HEXTORAW('41757468656e746963617465642062793a204441544142415345'),NULL,HEXTORAW('6f7261636c65'),NULL,NULL,NULL,HEXTORAW('c106'),NULL,HEXTORAW('787b0b16111330351d2b00'),NULL,NULL,HEXTORAW('80'),HEXTORAW('3735363037'),HEXTORAW('0000000000000000'),NULL,NULL,HEXTORAW('c515430b0123'),HEXTORAW(''),HEXTORAW(''),NULL);', csf=0, threadId=1, rbaLogFileSeq=2282, rbaBlockId=1, rbaByteOffset=424, rollback=0, info=Dictionary Mismatch, rsId='0x0008ea.00004303.01a8'}],
    @Boolean[true],
]
method=com.taobao.drc.logminer.log.OracleLogEntryAnalyzer.needFilterThisEntry location=AtExit
ts=2023-11-23 00:19:11; [cost=0.202728ms] result=@ArrayList[
    @OracleLogEntry[OracleLogEntry{rownum=30767, ownerId='UNKNOWN', tableId='OBJ# 383', scn=20032575, timestamp='2023-11-22 16:18:47', xid='1B000F002E0B0000', rowid='AAAAF/AABAAAV0oAAU', opType=UPDATE, segmentType=UNKNOWN, segmentName='OBJ# 383', redo='update "UNKNOWN"."OBJ# 383" set "COL 40" = NULL, "COL 41" = NULL where "COL 1" = HEXTORAW('c3075553') and "COL 2" = HEXTORAW('c102') and "COL 3" = HEXTORAW('c102') and "COL 4" IS NULL and "COL 5" = HEXTORAW('4f4254455354') and "COL 6" = HEXTORAW('643333653861323734356665') and "COL 7" = HEXTORAW('7074732f3234') and "COL 8" = HEXTORAW('c202') and "COL 9" = HEXTORAW('80') and "COL 10" IS NULL and "COL 11" IS NULL and "COL 12" IS NULL and "COL 13" IS NULL and "COL 14" IS NULL and "COL 15" IS NULL and "COL 16" IS NULL and "COL 17" IS NULL and "COL 18" IS NULL and "COL 19" IS NULL and "COL 20" IS NULL and "COL 21" IS NULL and "COL 22" IS NULL and "COL 23" = HEXTORAW('41757468656e746963617465642062793a204441544142415345') and "COL 24" IS NULL and "COL 25" = HEXTORAW('6f7261636c65') and "COL 26" IS NULL and "COL 27" IS NULL and "COL 28" IS NULL and "COL 29" = HEXTORAW('c106') and "COL 30" IS NULL and "COL 31" = HEXTORAW('787b0b16111330351d2b00') and "COL 32" IS NULL and "COL 33" IS NULL and "COL 34" = HEXTORAW('80') and "COL 35" = HEXTORAW('3735363037') and "COL 36" = HEXTORAW('0000000000000000') and "COL 37" IS NULL and "COL 38" IS NULL and "COL 39" = HEXTORAW('c515430b0123') and ROWID = 'AAAAF/AABAAAV0oAAU';', csf=0, threadId=1, rbaLogFileSeq=2282, rbaBlockId=1, rbaByteOffset=432, rollback=0, info=Dictionary Mismatch, rsId='0x0008ea.00004304.01b0'}],
    @Boolean[true],
]
method=com.taobao.drc.logminer.log.OracleLogEntryAnalyzer.needFilterThisEntry location=AtExit
ts=2023-11-23 00:19:11; [cost=0.01347ms] result=@ArrayList[
    @OracleLogEntry[OracleLogEntry{rownum=30770, ownerId='UNKNOWN', tableId='OBJ# 73645', scn=20032582, timestamp='2023-11-22 16:18:53', xid='18000D003B070000', rowid='AAASNBAAAAAAAAAAAA', opType=INSERT, segmentType=UNKNOWN, segmentName='OBJ# 73645', redo='insert into "UNKNOWN"."OBJ# 73645"("COL 1","COL 2","COL 3","COL 4","COL 5","COL 6","COL 7","COL 8","COL 9","COL 10") values (HEXTORAW('c20a64'),HEXTORAW('c20a64'),HEXTORAW('c20a64'),HEXTORAW('c102'),HEXTORAW('c20a64'),NULL,HEXTORAW('80'),HEXTORAW('c20a64'),HEXTORAW('c102'),HEXTORAW('612020202020202020202020202020202020202020202020'));', csf=0, threadId=1, rbaLogFileSeq=2282, rbaBlockId=1, rbaByteOffset=16, rollback=0, info=Dictionary Mismatch, rsId='0x0008ea.00004307.0010'}],
    @Boolean[true],
]

``` 

诊断: 

```
SQL> select * from all_objects where object_id = 73645;

OWNER			       OBJECT_NAME
------------------------------ ------------------------------
SUBOBJECT_NAME			OBJECT_ID DATA_OBJECT_ID OBJECT_TYPE
------------------------------ ---------- -------------- -------------------
CREATED   LAST_DDL_ TIMESTAMP		STATUS	T G S  NAMESPACE
--------- --------- ------------------- ------- - - - ----------
EDITION_NAME
------------------------------
OBTEST			       IORDL
				    73645	   74561 INDEX
16-NOV-23 22-NOV-23 2023-11-16:10:15:28 VALID	N N N	       4
 
 
---
 
定义: 
SQL> SET LONG 10000
SET LONGCHUNKSIZE 1000
SET LINESIZE 1000
SET PAGESIZE 0
SET TRIMOUT ON
SET TRIMSPOOL ON
SELECT
  dbms_metadata.get_ddl('INDEX', 'IORDL', 'OBTEST') AS index_definition
FROM
  dual;SQL> SQL> SQL> SQL> SQL> SQL>   2    3    4

  CREATE UNIQUE INDEX "OBTEST"."IORDL" ON "OBTEST"."ORDER_LINE" ("OL_W_ID", "OL_D_ID", "OL_O_ID", "OL_NUMBER")
  PCTFREE 10 INITRANS 4 MAXTRANS 255 COMPUTE STATISTICS
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1 BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "OBTEST"

``` 

查看TPCC各个表的定义, 仅出问题的两张表有主键, 且主键是联合主键: 

```
obclient [OBTEST]> show create table HISTORY\G
*************************** 1. row ***************************
       TABLE: HISTORY
CREATE TABLE: CREATE TABLE "HISTORY" (
  "H_C_ID" NUMBER,
  "H_C_D_ID" NUMBER,
  "H_C_W_ID" NUMBER,
  "H_D_ID" NUMBER,
  "H_W_ID" NUMBER,
  "H_DATE" DATE,
  "H_AMOUNT" NUMBER(6,2),
  "H_DATA" VARCHAR2(24),
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "UK_HISTORY_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.008 sec)

obclient [OBTEST]> show create table ITEM\G
*************************** 1. row ***************************
       TABLE: ITEM
CREATE TABLE: CREATE TABLE "ITEM" (
  "I_ID" NUMBER(6),
  "I_IM_ID" NUMBER,
  "I_NAME" VARCHAR2(24),
  "I_PRICE" NUMBER(5,2),
  "I_DATA" VARCHAR2(50),
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "ITEM_I1" UNIQUE ("I_ID"),
  CONSTRAINT "UK_ITEM_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.031 sec)

obclient [OBTEST]> show create table CUSTOMER\G
*************************** 1. row ***************************
       TABLE: CUSTOMER
CREATE TABLE: CREATE TABLE "CUSTOMER" (
  "C_ID" NUMBER(5),
  "C_D_ID" NUMBER(2),
  "C_W_ID" NUMBER(6),
  "C_FIRST" VARCHAR2(16),
  "C_MIDDLE" CHAR(2),
  "C_LAST" VARCHAR2(16),
  "C_STREET_1" VARCHAR2(20),
  "C_STREET_2" VARCHAR2(20),
  "C_CITY" VARCHAR2(20),
  "C_STATE" CHAR(2),
  "C_ZIP" CHAR(9),
  "C_PHONE" CHAR(16),
  "C_SINCE" DATE,
  "C_CREDIT" CHAR(2),
  "C_CREDIT_LIM" NUMBER(12,2),
  "C_DISCOUNT" NUMBER(4,4),
  "C_BALANCE" NUMBER(12,2),
  "C_YTD_PAYMENT" NUMBER(12,2),
  "C_PAYMENT_CNT" NUMBER(8),
  "C_DELIVERY_CNT" NUMBER(8),
  "C_DATA" VARCHAR2(500),
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "CUSTOMER_I1" UNIQUE ("C_W_ID", "C_D_ID", "C_ID"),
  CONSTRAINT "CUSTOMER_I2" UNIQUE ("C_LAST", "C_W_ID", "C_D_ID", "C_FIRST", "C_ID"),
  CONSTRAINT "UK_CUSTOMER_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.007 sec)

obclient [OBTEST]> show create table ORDERS\G
*************************** 1. row ***************************
       TABLE: ORDERS
CREATE TABLE: CREATE TABLE "ORDERS" (
  "O_ID" NUMBER,
  "O_W_ID" NUMBER,
  "O_D_ID" NUMBER,
  "O_C_ID" NUMBER,
  "O_CARRIER_ID" NUMBER,
  "O_OL_CNT" NUMBER,
  "O_ALL_LOCAL" NUMBER,
  "O_ENTRY_D" DATE,
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "ORDERS_I1" UNIQUE ("O_W_ID", "O_D_ID", "O_ID"),
  CONSTRAINT "ORDERS_I2" UNIQUE ("O_W_ID", "O_D_ID", "O_C_ID", "O_ID"),
  CONSTRAINT "UK_ORDERS_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.047 sec)

obclient [OBTEST]> show create table ORDER_LINE\G
*************************** 1. row ***************************
       TABLE: ORDER_LINE
CREATE TABLE: CREATE TABLE "ORDER_LINE" (
  "OL_W_ID" NUMBER CONSTRAINT "ORDER_LINE_OBNOTNULL_1700144717278591" NOT NULL ENABLE,
  "OL_D_ID" NUMBER CONSTRAINT "ORDER_LINE_OBNOTNULL_1700144717278628" NOT NULL ENABLE,
  "OL_O_ID" NUMBER CONSTRAINT "ORDER_LINE_OBNOTNULL_1700144717278645" NOT NULL ENABLE,
  "OL_NUMBER" NUMBER CONSTRAINT "ORDER_LINE_OBNOTNULL_1700144717278670" NOT NULL ENABLE,
  "OL_I_ID" NUMBER,
  "OL_DELIVERY_D" DATE,
  "OL_AMOUNT" NUMBER,
  "OL_SUPPLY_W_ID" NUMBER,
  "OL_QUANTITY" NUMBER,
  "OL_DIST_INFO" CHAR(24),
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "IORDL" PRIMARY KEY ("OL_W_ID", "OL_D_ID", "OL_O_ID", "OL_NUMBER"),
  CONSTRAINT "UK_ORDER_LINE_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.009 sec)

obclient [OBTEST]> show create table NEW_ORDER\G
*************************** 1. row ***************************
       TABLE: NEW_ORDER
CREATE TABLE: CREATE TABLE "NEW_ORDER" (
  "NO_W_ID" NUMBER CONSTRAINT "NEW_ORDER_OBNOTNULL_1700144682488703" NOT NULL ENABLE,
  "NO_D_ID" NUMBER CONSTRAINT "NEW_ORDER_OBNOTNULL_1700144682489740" NOT NULL ENABLE,
  "NO_O_ID" NUMBER CONSTRAINT "NEW_ORDER_OBNOTNULL_1700144682489760" NOT NULL ENABLE,
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "INORD" PRIMARY KEY ("NO_W_ID", "NO_D_ID", "NO_O_ID"),
  CONSTRAINT "UK_NEW_ORDER_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.028 sec)

obclient [OBTEST]> show create table WAREHOUSE\G

*************************** 1. row ***************************
       TABLE: WAREHOUSE
CREATE TABLE: CREATE TABLE "WAREHOUSE" (
  "W_ID" NUMBER(6),
  "W_YTD" NUMBER(12,2),
  "W_TAX" NUMBER(4,4),
  "W_NAME" VARCHAR2(10),
  "W_STREET_1" VARCHAR2(20),
  "W_STREET_2" VARCHAR2(20),
  "W_CITY" VARCHAR2(20),
  "W_STATE" CHAR(2),
  "W_ZIP" CHAR(9),
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "WAREHOUSE_I1" UNIQUE ("W_ID"),
  CONSTRAINT "UK_WAREHOUSE_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.195 sec)

obclient [OBTEST]> show create table STOCK\G
*************************** 1. row ***************************
       TABLE: STOCK
CREATE TABLE: CREATE TABLE "STOCK" (
  "S_I_ID" NUMBER(6),
  "S_W_ID" NUMBER(6),
  "S_QUANTITY" NUMBER(6),
  "S_DIST_01" CHAR(24),
  "S_DIST_02" CHAR(24),
  "S_DIST_03" CHAR(24),
  "S_DIST_04" CHAR(24),
  "S_DIST_05" CHAR(24),
  "S_DIST_06" CHAR(24),
  "S_DIST_07" CHAR(24),
  "S_DIST_08" CHAR(24),
  "S_DIST_09" CHAR(24),
  "S_DIST_10" CHAR(24),
  "S_YTD" NUMBER(10),
  "S_ORDER_CNT" NUMBER(6),
  "S_REMOTE_CNT" NUMBER(6),
  "S_DATA" VARCHAR2(50),
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "STOCK_I1" UNIQUE ("S_I_ID", "S_W_ID"),
  CONSTRAINT "UK_STOCK_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.016 sec)

obclient [OBTEST]> show create table DISTRICT\G
*************************** 1. row ***************************
       TABLE: DISTRICT
CREATE TABLE: CREATE TABLE "DISTRICT" (
  "D_ID" NUMBER(2),
  "D_W_ID" NUMBER(6),
  "D_YTD" NUMBER(12,2),
  "D_TAX" NUMBER(4,4),
  "D_NEXT_O_ID" NUMBER,
  "D_NAME" VARCHAR2(10),
  "D_STREET_1" VARCHAR2(20),
  "D_STREET_2" VARCHAR2(20),
  "D_CITY" VARCHAR2(20),
  "D_STATE" CHAR(2),
  "D_ZIP" CHAR(9),
  "OMS_OBJECT_NUMBER" NUMBER INVISIBLE,
  "OMS_RELATIVE_FNO" NUMBER INVISIBLE,
  "OMS_BLOCK_NUMBER" NUMBER INVISIBLE,
  "OMS_ROW_NUMBER" NUMBER INVISIBLE,
  CONSTRAINT "DISTRICT_I1" UNIQUE ("D_W_ID", "D_ID"),
  CONSTRAINT "UK_DISTRICT_OMS_ROWID" UNIQUE ("OMS_OBJECT_NUMBER", "OMS_RELATIVE_FNO", "OMS_BLOCK_NUMBER", "OMS_ROW_NUMBER")
) COMPRESS FOR ARCHIVE REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
1 row in set (0.004 sec)

``` 

使用手工复现方式, 无法复现出类似情况

在Oracle support中未找到类似的bug

收尾: 提议在OMS中检测类似情况:

  - 检测到Object ID是表的主键, 进行告警提示

  - 对记录忽略机制 (needFilterThisEntry函数) 的情况, 进行明确分类 (内部表, 辅助索引, 等), 对于位置分类进行告警提示. 增加诊断日志

  - 检测到Object ID是表的主键, 识别成表, 使得记录可以正确执行

再次诊断, 抓包 hammerdb发往Oracle的SQL: 

```
CREATE TABLE CUSTOMER (C_ID NUMBER(5, 0), C_D_ID NUMBER(2, 0), C_W_ID NUMBER(6, 0), C_FIRST VARCHAR2(16), C_MIDDLE CHAR(2), C_LAST VARCHAR2(16), C_STREET_1 VARCHAR2(20), C_STREET_2 VARCHAR2(20), C_CITY VARCHAR2(20), C_STATE CHAR(2), C_ZIP CHAR(9), C_PHONE. CHAR(16), C_SINCE DATE, C_CREDIT CHAR(2), C_CREDIT_LIM NUMBER(12, 2), C_DISCOUNT NUMBER(4, 4), C_BALANCE NUMBER(12, 2), C_YTD_PAYMENT NUMBER(12, 2), C_PAYMENT_CNT NUMBER(8, 0), C_DELIVERY_CNT NUMBER(8, 0), C_DATA VARCHAR2(500)) INITRANS 4 PCTFREE 10....................................................................................................................
...................................6...............`..
............................................................6ORA-00955: name is already used by an existing object
...........^	!...............E...........
.................................................................................................................................................CREATE TABLE DISTRICT (D_ID NUMBER(2, 0), D_W_ID NUMBER(6, 0), D_YTD NUMBER(12, 2), D_TAX NUMBER(4, 4), D_NEXT_O_ID NUMBER, D_NAME VARCHAR2(10), D_STREET_1 VARCHAR2(20), D_STREET_2 VARCHAR2(20), D_CITY VARCHAR2(20), D_STATE CHAR(2), D_ZIP CHAR(9)) INITRAN.S 4 PCTFREE 99 PCTUSED 1....................................................................................................................
............................	......6...............`..
............................................................6ORA-00955: name is already used by an existing object
...........^
!...........................
................................................................................................................................................CREATE TABLE HISTORY (H_C_ID NUMBER, H_C_D_ID NUMBER, H_C_W_ID NUMBER, H_D_ID NUMBER, H_W_ID NUMBER, H_DATE DATE, H_AMOUNT NUMBER(6, 2), H_DATA VARCHAR2(24)) INITRANS 4 PCTFREE 10....................................................................................................	..............
............................
......6...............`..
............................................................6ORA-00955: name is already used by an existing object
.y.........^.!...........................
................................................................................................................................................CREATE TABLE ITEM (I_ID NUMBER(6, 0), I_IM_ID NUMBER, I_NAME VARCHAR2(24), I_PRICE NUMBER(5, 2), I_DATA VARCHAR2(50)) INITRANS 4 PCTFREE 10....................................................................................................
..............
...................................6...............`..
............................................................6ORA-00955: name is already used by an existing object
...........^.!...........................
................................................................................................................................................CREATE TABLE WAREHOUSE (W_ID NUMBER(6, 0), W_YTD NUMBER(12, 2), W_TAX NUMBER(4, 4), W_NAME VARCHAR2(10), W_STREET_1 VARCHAR2(20), W_STREET_2 VARCHAR2(20), W_CITY VARCHAR2(20), W_STATE CHAR(2), W_ZIP CHAR(9)) INITRANS 4 PCTFREE 99 PCTUSED 1...................................................................................................................
...................................6...............`..
............................................................6ORA-00955: name is already used by an existing object
...........^
!...........................
.................................................................................................................................................CREATE TABLE STOCK (S_I_ID NUMBER(6, 0), S_W_ID NUMBER(6, 0), S_QUANTITY NUMBER(6, 0), S_DIST_01 CHAR(24), S_DIST_02 CHAR(24), S_DIST_03 CHAR(24), S_DIST_04 CHAR(24), S_DIST_05 CHAR(24), S_DIST_06 CHAR(24), S_DIST_07 CHAR(24), S_DIST_08 CHAR(24), S_DIST_0.9 CHAR(24), S_DIST_10 CHAR(24), S_YTD NUMBER(10, 0), S_ORDER_CNT NUMBER(6, 0), S_REMOTE_CNT NUMBER(6, 0), S_DATA VARCHAR2(50)) INITRANS 4 PCTFREE 10....................................................................................................................
............................
......6...............`..
............................................................6ORA-00955: name is already used by an existing object
...........^.!...............7...........
................................................................................................................................................CREATE TABLE NEW_ORDER (NO_W_ID NUMBER, NO_D_ID NUMBER, NO_O_ID NUMBER, CONSTRAINT INORD PRIMARY KEY (NO_W_ID, NO_D_ID, NO_O_ID) ENABLE ) ORGANIZATION INDEX NOCOMPRESS INITRANS 4 PCTFREE 10....................................................................................................
..............
...................................6...............`..
............................................................6ORA-00955: name is already used by an existing object
...........^.!...............
...........
................................................................................................................................................CREATE TABLE ORDERS (O_ID NUMBER, O_W_ID NUMBER, O_D_ID NUMBER, O_C_ID NUMBER, O_CARRIER_ID NUMBER, O_OL_CNT NUMBER, O_ALL_LOCAL NUMBER, O_ENTRY_D DATE) INITRANS 4 PCTFREE 10...................................................................................................................
...................................6...............`..
............................................................6ORA-00955: name is already used by an existing object
.C.........^.!...........................
.................................................................................................................................................CREATE TABLE ORDER_LINE (OL_W_ID NUMBER, OL_D_ID NUMBER, OL_O_ID NUMBER, OL_NUMBER NUMBER, OL_I_ID NUMBER, OL_DELIVERY_D DATE, OL_AMOUNT NUMBER, OL_SUPPLY_W_ID NUMBER, OL_QUANTITY NUMBER, OL_DIST_INFO CHAR(24), CONSTRAINT IORDL PRIMARY KEY (OL_W_ID, OL_D_SID, OL_O_ID, OL_NUMBER) ENABLE) ORGANIZATION INDEX NOCOMPRESS INITRANS 4 PCTFREE 10....................................................................................................................
...................................6...............`..
............................................................6ORA-00955: name is already used by an existing object
...........^.!...............6...........
.................................................................................................................................................CREATE OR REPLACE VIEW STOCK_ITEM (I_ID, S_W_ID, I_PRICE, I_NAME, I_DATA, S_DATA, S_QUANTITY, S_ORDER_CNT, S_YTD, S_REMOTE_CNT, S_DIST_01, S_DIST_02, S_DIST_03, S_DIST_04, S_DIST_05, S_DIST_06, S_DIST_07, S_DIST_08, S_DIST_09, S_DIST_10) AS SELECT /*+ LEA.DING(S) USE_NL(I) */ I.I_ID, S_W_ID, I.I_PRICE, I.I_NAME, I.I_DATA, S_DATA, S_QUANTITY, S_ORDER_CNT, S_YTD, S_REMOTE_CNT, S_DIST_01, S_DIST_02, S_DIST_03, S_DIST_04, S_DIST_05, S_DIST_06, S_DIST_07, S_DIST_08, S_DIST_09, S_DIST_10 FROM STOCK S, ITEM I WHE.RE I.I_ID = S.S_I_ID..................................................................n.>.....................................................................................6...............`..
``` 

取出语句, 对比测试: 

```
CREATE TABLE ORDER_LINE_1 (OL_W_ID NUMBER, OL_D_SID NUMBER, OL_O_ID NUMBER, OL_NUMBER NUMBER, OL_I_ID NUMBER, OL_DELIVERY_D DATE, OL_AMOUNT NUMBER, OL_SUPPLY_W_ID NUMBER, OL_QUANTITY NUMBER, OL_DIST_INFO CHAR(24), CONSTRAINT IORDL PRIMARY KEY (OL_W_ID, OL_D_SID, OL_O_ID, OL_NUMBER) ENABLE) ORGANIZATION INDEX NOCOMPRESS INITRANS 4 PCTFREE 10;
INSERT INTO order_line_1 (ol_o_id, ol_d_sid, ol_w_id, ol_number, ol_i_id, ol_supply_w_id, ol_quantity, ol_amount, ol_dist_info, ol_delivery_d) values (1999, 999, 999, 1, 999, 999, 1, 0, 'a', NULL);
# 无法复制

CREATE TABLE ORDER_LINE_2 (OL_W_ID NUMBER, OL_D_SID NUMBER, OL_O_ID NUMBER, OL_NUMBER NUMBER, OL_I_ID NUMBER, OL_DELIVERY_D DATE, OL_AMOUNT NUMBER, OL_SUPPLY_W_ID NUMBER, OL_QUANTITY NUMBER, OL_DIST_INFO CHAR(24), CONSTRAINT IORDL_2 PRIMARY KEY (OL_W_ID, OL_D_SID, OL_O_ID, OL_NUMBER) ENABLE) ;
INSERT INTO order_line_2 (ol_o_id, ol_d_sid, ol_w_id, ol_number, ol_i_id, ol_supply_w_id, ol_quantity, ol_amount, ol_dist_info, ol_delivery_d) values (1999, 999, 999, 1, 999, 999, 1, 0, 'a', NULL);
# 可以复制
 
CREATE TABLE ORDER_LINE_4 (OL_W_ID NUMBER, OL_D_SID NUMBER, OL_O_ID NUMBER, OL_NUMBER NUMBER, OL_I_ID NUMBER, OL_DELIVERY_D DATE, OL_AMOUNT NUMBER, OL_SUPPLY_W_ID NUMBER, OL_QUANTITY NUMBER, OL_DIST_INFO CHAR(24), CONSTRAINT IORDL_4 PRIMARY KEY (OL_W_ID, OL_D_SID, OL_O_ID, OL_NUMBER) ENABLE) ORGANIZATION INDEX;
INSERT INTO order_line_4 (ol_o_id, ol_d_sid, ol_w_id, ol_number, ol_i_id, ol_supply_w_id, ol_quantity, ol_amount, ol_dist_info, ol_delivery_d) values (1999, 999, 999, 1, 999, 999, 1, 0, 'a', NULL);
# 确认IoT表会丢失数据
``` 

### TODO

事务大小: 

```
[arthas@40990]$ watch com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink offer 'params[0].size()' -n 100
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 67 ms, listenerId: 7
method=com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer location=AtExit
ts=2023-11-21 17:58:49; [cost=97.825255ms] result=@Integer[128]
method=com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer location=AtExit
ts=2023-11-21 17:58:49; [cost=125.011576ms] result=@Integer[128]
method=com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer location=AtExit
ts=2023-11-21 17:58:49; [cost=85.671744ms] result=@Integer[128]
method=com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer location=AtExit
ts=2023-11-21 17:58:49; [cost=103.166414ms] result=@Integer[128]
method=com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer location=AtExit
ts=2023-11-21 17:58:49; [cost=117.070951ms] result=@Integer[128]
``` 

OBOracleJDBCSink.offer 会有明显的卡顿周期

还发现数据数量不一致, 待复现 (先调大oracle的archive log阈值)
