---
title: 20231112 - OMS, 使用arthas将其处理队列中的元素导出
confluence_page_id: 2589451
created_at: 2023-11-11T16:54:35+00:00
updated_at: 2023-11-14T15:54:07+00:00
---

# 导出队列1的积压

```
vmtool --action getInstances --className com.taobao.drc.logminer.OracleRacLogRecordGenerator --limit 100 --express '
#generator=instances.{? #this.context.logmnrDataSource.url == "10.186.16.126:1521/EE.ORACLE.DOCKER" && #this.context.logmnrDataSource.username == "OBTEST"}[0],
#fetcher=#generator.fetchers[0],
#fetcher_OutQueue=#fetcher.queue.queue.toArray(),
#fetcher_OutQueue.{#this.toString()}'
``` 

样例输出: 

```
@ArrayList[
    @String[OracleLogEntry{rownum=9449, ownerId='UNKNOWN', tableId='OBJ# 73640', scn=11568453, timestamp='2023-11-11 16:51:26', xid='02001B00FF060000', rowid='AAAR+oAAFAAAARfACG', opType=DELETE, segmentType=UNKNOWN, segmentName='OBJ# 73640', redo='delete from "UNKNOWN"."OBJ# 73640" where "COL 1" = HEXTORAW('c30a3026') and "COL 2" = HEXTORAW('4669727374204e616d65203337') and "COL 3" = HEXTORAW('4c617374204e616d65203337') and "COL 4" = HEXTORAW('787b0b08062501') and "COL 5" = HEXTORAW('c30947') and ROWID = 'AAAR+oAAFAAAARfACG';', csf=0, threadId=1, rbaLogFileSeq=679, rbaBlockId=1, rbaByteOffset=232, rollback=0, info=Dictionary Mismatch, rsId='0x0002a7.000013f5.00e8'}],
    @String[OracleLogEntry{rownum=9450, ownerId='null', tableId='OBJ# 73641', scn=11568453, timestamp='2023-11-11 16:51:26', xid='02001B00FF060000', rowid='AAAR+pAAAAAAAAAAAA', opType=INTERNAL, segmentType=UNKNOWN, segmentName='null', redo='null', csf=0, threadId=1, rbaLogFileSeq=679, rbaBlockId=1, rbaByteOffset=52, rollback=0, info=null, rsId='0x0002a7.000013f6.0034'}],
    @String[OracleLogEntry{rownum=9451, ownerId='UNKNOWN', tableId='OBJ# 73640', scn=11568453, timestamp='2023-11-11 16:51:26', xid='02001B00FF060000', rowid='AAAR+oAAFAAAARfACH', opType=DELETE, segmentType=UNKNOWN, segmentName='OBJ# 73640', redo='delete from "UNKNOWN"."OBJ# 73640" where "COL 1" = HEXTORAW('c30a3027') and "COL 2" = HEXTORAW('4669727374204e616d65203338') and "COL 3" = HEXTORAW('4c617374204e616d65203338') and "COL 4" = HEXTORAW('787b0b08062501') and "COL 5" = HEXTORAW('c30951') and ROWID = 'AAAR+oAAFAAAARfACH';', csf=0, threadId=1, rbaLogFileSeq=679, rbaBlockId=1, rbaByteOffset=268, rollback=0, info=Dictionary Mismatch, rsId='0x0002a7.000013f6.010c'}],
...
...
...
]
``` 

# 监听各队列的入口

注意: 暂时不考虑多个RAC节点的情况

```
源端进程: 
 
options json-format true
watch com.taobao.drc.logminer.queue.OracleLogEntryQueue put '{"Queue-1", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[0]}' -b >> /tmp/watch_source.out &
watch com.taobao.drc.logminer.queue.OracleLogRecordQueue putLogRecord '{"Queue-2", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[0]}' -b  >> /tmp/watch_source.out &
watch com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator putRecordToQueue '{"Queue-3", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[0]}' -b >> /tmp/watch_source.out &
watch com.taobao.drc.logminer.LogRecordConverterPrepare$LogRecordsTrx nextLogRecord '{"Queue-4", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, returnObj}' 'returnObj != null' -s >> /tmp/watch_source.out &
watch com.taobao.drc.logminer.queue.OracleBackRecordQueue put '{"Queue-5", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[0]}' '!@java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name.equals("Record Sorter")' -b >> /tmp/watch_source.out &
watch com.taobao.drc.logminer.queue.OracleBackRecordQueue readyRecordNotBackQueried '{"Queue-6 (No Look-up)", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[0]}' -b >> /tmp/watch_source.out &
watch com.taobao.drc.logminer.queue.OracleBackRecordQueue readyRecordBackQueried '{"Queue-7.1 (Look-up done)", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[0]}' -b >> /tmp/watch_source.out &
watch com.taobao.drc.logminer.queue.OracleBackRecordQueue put '{"Queue-8", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[0]}' '@java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name.equals("Record Sorter")' -b >> /tmp/watch_source.out &
watch LogMinerConnectorConverter convert '{"Queue-9 (Store)", @java.lang.System@currentTimeMillis(), @java.lang.Thread@currentThread().name, params[1]}' -b -n 4 >> /tmp/watch_source.out &
```
```
目标端: 
 
watch com.oceanbase.oms.connector.batch.CommonTransactionAssembler notify '{"(From store)", @java.lang.Thread@currentThread().name, params[0]}' '!(params[0] instanceof com.oceanbase.oms.logmessage.DataMessage)' -b -x2 >> watch.out &
 
watch com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner putTransaction '{"Queue-11", @java.lang.Thread@currentThread().name, params[0]}' -s  >> watch.out &
 
watch com.oceanbase.connector.framework.threadmanager.bridgetask.SerialBridgeConnectorTask processRecordBatch '{"Queue-12", @java.lang.Thread@currentThread().name, returnObj}' 'returnObj != null' -s >> watch.out &
 
watch com.oceanbase.oms.connector.dispatcher.scheduler.DefaultTransactionScheduler submitTransaction '{"Queue-13", @java.lang.Thread@currentThread().name, params[0]}' -s -x2 >> watch.out &
 
watch com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink offer '{"(JDBC)", @java.lang.Thread@currentThread().name, params[0]}' -b -s -x2 >> watch.out &
``` 

问题: 使用批处理脚本时, 异步后台运行的功能不好用. 转向使用HTTP API

# 使用HTTP API

启动/停止 HTTP API接口: 

```
java -jar arthas-boot.jar -c "version" 14680
java -jar arthas-boot.jar -c "stop" 14680
 
 
curl -Ss -XPOST http://localhost:8563/api -d '
{
  "action":"init_session"
}
'

curl -Ss -XPOST http://localhost:8563/api -d '
{
  "action":"async_exec",
  "sessionId":"ed7e5c62-f3b9-4989-a0f0-40f22ab16756",
  "command":"watch com.taobao.drc.logminer.queue.OracleLogEntryQueue put '{@java.lang.System@currentTimeMillis\(\)\,1}'
}
'

curl -Ss -XPOST http://localhost:8563/api -d '
{
  "action":"join_session",
  "sessionId" : "ed7e5c62-f3b9-4989-a0f0-40f22ab16756"
}
'

curl -Ss -XPOST http://localhost:8563/api -d '
{
  "action":"pull_results",
  "sessionId" : "ed7e5c62-f3b9-4989-a0f0-40f22ab16756",
  "consumerId" : "ddd6687854434e90a94b52bc82a116e1_2"
}
'
``` 

使用python脚本, 调用HTTP API, <http://10.186.18.21/huangyan/oms-copilot>

# 源端结果

```
["Queue-1",1699932381122,"LogEntryPrimaryFetcher#1",{"epochSecondTimestamp":1699932369,"rsId":"0x000021.000013b0.0010"}]
["Queue-1",1699932381123,"LogEntryPrimaryFetcher#1",{"epochSecondTimestamp":1699932369,"rsId":"0x000021.000013b0.0010"}]
["Queue-1",1699932381123,"LogEntryPrimaryFetcher#1",{"epochSecondTimestamp":1699932369,"rsId":"0x000021.000013b1.0190"}]
["Queue-2",1699932381124,"Oracle Log Extractor Instance#1",{"rsid":"0x000021.000013b0.0010","threadid":1}]
["Queue-2",1699932381124,"Oracle Log Extractor Instance#1",{"rsid":"0x000021.000013b0.0010","threadid":1}]
["Queue-2",1699932381125,"Oracle Log Extractor Instance#1",{"rsid":"0x000021.000013b1.0190","threadid":1}]
["Queue-3",1699932381126,"Oracle LogRecord Serializer Instance#1",{"rsid":"0x000021.000013b0.0010","threadid":1}]
["Queue-3",1699932381127,"Oracle LogRecord Serializer Instance#1",{"rsid":"0x000021.000013b0.0010","threadid":1}]
["Queue-3",1699932381127,"Oracle LogRecord Serializer Instance#1",{"rsid":"0x000021.000013b1.0190","threadid":1}]
["Queue-4",1699932381127,"Oracle LogRecord Converter Prepare",{"rsid":"0x000021.000013b0.0010","threadid":1}]
["Queue-4",1699932381127,"Oracle LogRecord Converter Prepare",{"rsid":"0x000021.000013b0.0010","threadid":1}]
["Queue-4",1699932381128,"Oracle LogRecord Converter Prepare",{"rsid":"0x000021.000013b1.0190","threadid":1}]
["Queue-5",1699932381132,"Record Sorter",{"primariesInOrder":[],"type":"START","uniquesInOrder":[]}]
["Queue-5",1699932381133,"Record Sorter",{"primariesInOrder":[["EMPLOYEE_ID"]],"type":"INSERT","uniquesInOrder":[]}]
["Queue-5",1699932381133,"Record Sorter",{"primariesInOrder":[],"type":"COMMIT","uniquesInOrder":[]}]
["Queue-6 (No Look-up)",1699932381129,"Record Dispatcher",{"primariesInOrder":[],"type":"START","uniquesInOrder":[]}]
["Queue-6 (No Look-up)",1699932381131,"Record Dispatcher",{"primariesInOrder":[["EMPLOYEE_ID"]],"type":"INSERT","uniquesInOrder":[]}]
["Queue-6 (No Look-up)",1699932381131,"Record Dispatcher",{"primariesInOrder":[],"type":"COMMIT","uniquesInOrder":[]}]
["Queue-8",1699932381132,"Record Sorter",{"primariesInOrder":[],"type":"START","uniquesInOrder":[]}]
["Queue-8",1699932381133,"Record Sorter",{"primariesInOrder":[["EMPLOYEE_ID"]],"type":"INSERT","uniquesInOrder":[]}]
["Queue-8",1699932381134,"Record Sorter",{"primariesInOrder":[],"type":"COMMIT","uniquesInOrder":[]}]
["Queue-9 (Store)",1699932381235,"pool-2-thread-1",{}]
["Queue-9 (Store)",1699932381235,"pool-2-thread-1",{"postColumns":[{"encoding":"UTF-8","index":0,"name":"EMPLOYEE_ID","notNull":true,"pK":true,"type":2,"uK":false,"valOrigin":0,"value":"303","valueLength":3},{"encoding":"UTF-8","index":1,"name":"FIRST_NAME","notNull":false,"pK":false,"type":3,"uK":false,"valOrigin":0,"value":"v_first_name","valueLength":12},{"encoding":"UTF-8","index":2,"name":"LAST_NAME","notNull":false,"pK":false,"type":3,"uK":false,"valOrigin":0,"value":"v_last_name","valueLength":11},{"encoding":"UTF-8","index":3,"name":"HIRE_DATE","notNull":false,"pK":false,"type":10,"uK":false,"valOrigin":0,"valueLength":-1},{"encoding":"UTF-8","index":4,"name":"JOB_TITLE","notNull":false,"pK":false,"type":3,"uK":false,"valOrigin":0,"value":"v_job_title","valueLength":11},{"encoding":"UTF-8","index":5,"name":"SALARY","notNull":false,"pK":false,"type":2,"uK":false,"valOrigin":0,"value":"1000","valueLength":4},{"encoding":"UTF-8","index":6,"name":"rowid","notNull":true,"pK":false,"type":8,"uK":false,"valOrigin":0,"value":"AAAR7QAAFAAAACGAAS","valueLength":18}],"primariesInOrder":[["EMPLOYEE_ID"]],"uniquesInOrder":[]}]
["Queue-9 (Store)",1699932381235,"pool-2-thread-1",{}]
["Queue-4",1699932401571,"Oracle LogRecord Converter Prepare",{"rsid":"0x000021.000013b3.0010","threadid":1}]
["Queue-6 (No Look-up)",1699932401572,"Record Dispatcher",{"primariesInOrder":[],"type":"START","uniquesInOrder":[]}]
["Queue-5",1699932401574,"Record Sorter",{"primariesInOrder":[],"type":"START","uniquesInOrder":[]}]
["Queue-8",1699932401574,"Record Sorter",{"primariesInOrder":[],"type":"START","uniquesInOrder":[]}]
["Queue-6 (No Look-up)",1699932401573,"Record Dispatcher",{"primariesInOrder":[["EMPLOYEE_ID"]],"type":"INSERT","uniquesInOrder":[]}]
["Queue-6 (No Look-up)",1699932401573,"Record Dispatcher",{"primariesInOrder":[],"type":"COMMIT","uniquesInOrder":[]}]
["Queue-5",1699932401574,"Record Sorter",{"primariesInOrder":[["EMPLOYEE_ID"]],"type":"INSERT","uniquesInOrder":[]}]
["Queue-4",1699932401572,"Oracle LogRecord Converter Prepare",{"rsid":"0x000021.000013b3.0010","threadid":1}]
["Queue-8",1699932401575,"Record Sorter",{"primariesInOrder":[["EMPLOYEE_ID"]],"type":"INSERT","uniquesInOrder":[]}]
["Queue-5",1699932401575,"Record Sorter",{"primariesInOrder":[],"type":"COMMIT","uniquesInOrder":[]}]
["Queue-8",1699932401576,"Record Sorter",{"primariesInOrder":[],"type":"COMMIT","uniquesInOrder":[]}]
["Queue-4",1699932401572,"Oracle LogRecord Converter Prepare",{"rsid":"0x000021.000013b4.0160","threadid":1}]
["Queue-9 (Store)",1699932401676,"pool-2-thread-1",{}]
["Queue-9 (Store)",1699932401676,"pool-2-thread-1",{"postColumns":[{"encoding":"UTF-8","index":0,"name":"EMPLOYEE_ID","notNull":true,"pK":true,"type":2,"uK":false,"valOrigin":0,"value":"304","valueLength":3},{"encoding":"UTF-8","index":1,"name":"FIRST_NAME","notNull":false,"pK":false,"type":3,"uK":false,"valOrigin":0,"value":"v_first_name","valueLength":12},{"encoding":"UTF-8","index":2,"name":"LAST_NAME","notNull":false,"pK":false,"type":3,"uK":false,"valOrigin":0,"value":"v_last_name","valueLength":11},{"encoding":"UTF-8","index":3,"name":"HIRE_DATE","notNull":false,"pK":false,"type":10,"uK":false,"valOrigin":0,"valueLength":-1},{"encoding":"UTF-8","index":4,"name":"JOB_TITLE","notNull":false,"pK":false,"type":3,"uK":false,"valOrigin":0,"value":"v_job_title","valueLength":11},{"encoding":"UTF-8","index":5,"name":"SALARY","notNull":false,"pK":false,"type":2,"uK":false,"valOrigin":0,"value":"1000","valueLength":4},{"encoding":"UTF-8","index":6,"name":"rowid","notNull":true,"pK":false,"type":8,"uK":false,"valOrigin":0,"value":"AAAR7QAAFAAAACGAAT","valueLength":18}],"primariesInOrder":[["EMPLOYEE_ID"]],"uniquesInOrder":[]}]
["Queue-9 (Store)",1699932401677,"pool-2-thread-1",{}]
["Queue-3",1699932401570,"Oracle LogRecord Serializer Instance#1",{"rsid":"0x000021.000013b3.0010","threadid":1}]
["Queue-1",1699932401568,"LogEntryPrimaryFetcher#1",{"epochSecondTimestamp":1699932380,"rsId":"0x000021.000013b3.0010"}]
["Queue-3",1699932401571,"Oracle LogRecord Serializer Instance#1",{"rsid":"0x000021.000013b3.0010","threadid":1}]
["Queue-3",1699932401571,"Oracle LogRecord Serializer Instance#1",{"rsid":"0x000021.000013b4.0160","threadid":1}]
["Queue-1",1699932401569,"LogEntryPrimaryFetcher#1",{"epochSecondTimestamp":1699932380,"rsId":"0x000021.000013b3.0010"}]
["Queue-1",1699932401569,"LogEntryPrimaryFetcher#1",{"epochSecondTimestamp":1699932380,"rsId":"0x000021.000013b4.0160"}]
["Queue-2",1699932401569,"Oracle Log Extractor Instance#1",{"rsid":"0x000021.000013b3.0010","threadid":1}]
["Queue-2",1699932401570,"Oracle Log Extractor Instance#1",{"rsid":"0x000021.000013b3.0010","threadid":1}]
["Queue-2",1699932401570,"Oracle Log Extractor Instance#1",{"rsid":"0x000021.000013b4.0160","threadid":1}]

``` 

问题: 

  - 队列元素的ID 不一致, 无法明确某一个记录的路径, 需要增加日志
  - 0x000021.000013b3.0010, 会在处理完后再次出现, 怀疑和logminer的获取块的间隙重复有关

增加输出内容: 

```
bash-4.2$ cat watch.log
{"timestamp": 1699945110664, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "32346", "scn": "1371665", "rowid": "null", "timestamp": "1699945098000", "type": "START", "table": "null", "transaction": "0A001F00D5060000", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003466.0010", "threadid": "1"}, "rsid": "0x000021.00003466.0010", "scn": "1371665", "xid": ""}
{"timestamp": 1699945110666, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "32347", "scn": "1371665", "rowid": "AAAR7QAAFAAAACGAAk", "timestamp": "1699945098000", "type": "INSERT", "table": "73424", "transaction": "0A001F00D5060000", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003466.0010", "threadid": "1"}, "rsid": "0x000021.00003466.0010", "scn": "1371665", "xid": ""}
{"timestamp": 1699945110667, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "32348", "scn": "1371666", "rowid": "null", "timestamp": "1699945098000", "type": "COMMIT", "table": "null", "transaction": "0A001F00D5060000", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003467.0190", "threadid": "1"}, "rsid": "0x000021.00003467.0190", "scn": "1371666", "xid": ""}
{"timestamp": 1699945110668, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "32346", "scn": "1371666", "rowid": "null", "timestamp": "1699945098000", "type": "START", "table": "null", "transaction": "1.0x000021.00003467.0190", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003466.0010", "threadid": "1"}, "rsid": "0x000021.00003466.0010", "scn": "1371666", "xid": "1.0x000021.00003467.0190"}
{"timestamp": 1699945110668, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "32347", "scn": "1371666", "rowid": "AAAR7QAAFAAAACGAAk", "timestamp": "1699945098000", "type": "INSERT", "table": "73424", "transaction": "1.0x000021.00003467.0190", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003466.0010", "threadid": "1"}, "rsid": "0x000021.00003466.0010", "scn": "1371666", "xid": "1.0x000021.00003467.0190"}
{"timestamp": 1699945110669, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "32348", "scn": "1371666", "rowid": "null", "timestamp": "1699945098000", "type": "COMMIT", "table": "null", "transaction": "1.0x000021.00003467.0190", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003467.0190", "threadid": "1"}, "rsid": "0x000021.00003467.0190", "scn": "1371666", "xid": "1.0x000021.00003467.0190"}
{"timestamp": 1699945110669, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "32346", "scn": "1371666", "rowid": "null", "timestamp": "1699945098000", "type": "START", "table": "null", "transaction": "1.0x000021.00003467.0190", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003466.0010", "threadid": "1"}, "rsid": "0x000021.00003466.0010", "scn": "1371666", "xid": "1.0x000021.00003467.0190"}
{"timestamp": 1699945110669, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "32347", "scn": "1371666", "rowid": "AAAR7QAAFAAAACGAAk", "timestamp": "1699945098000", "type": "INSERT", "table": "73424", "transaction": "1.0x000021.00003467.0190", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003466.0010", "threadid": "1"}, "rsid": "0x000021.00003466.0010", "scn": "1371666", "xid": "1.0x000021.00003467.0190"}
{"timestamp": 1699945110670, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "32348", "scn": "1371666", "rowid": "null", "timestamp": "1699945098000", "type": "COMMIT", "table": "null", "transaction": "1.0x000021.00003467.0190", "fileid": "33", "blockid": "1", "rollback": "0", "rsid": "0x000021.00003467.0190", "threadid": "1"}, "rsid": "0x000021.00003467.0190", "scn": "1371666", "xid": "1.0x000021.00003467.0190"}
{"timestamp": 1699945110671, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"scn": "1371666 ", "xid": "1.0x000021.00003467.0190 ", "timestamp": "1699945098 ", "type": "START }"}, "rsid": "", "scn": "1371666 ", "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110673, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"rowid": "AAAR7QAAFAAAACGAAk ", "xid": "1.0x000021.00003467.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699945098 ", "pkOrUk": "EMPLOYEE_ID=321 }"}, "rsid": "", "scn": null, "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110673, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"scn": "1371666 ", "xid": "1.0x000021.00003467.0190 ", "timestamp": "1699945098 ", "type": "COMMIT }"}, "rsid": "", "scn": "1371666 ", "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110673, "queue_name": "Queue-5", "thread_name": "Record Sorter", "data": {"scn": "1371666 ", "xid": "1.0x000021.00003467.0190 ", "timestamp": "1699945098 ", "type": "START }"}, "rsid": "", "scn": "1371666 ", "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110674, "queue_name": "Queue-5", "thread_name": "Record Sorter", "data": {"rowid": "AAAR7QAAFAAAACGAAk ", "xid": "1.0x000021.00003467.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699945098 ", "pkOrUk": "EMPLOYEE_ID=321 }"}, "rsid": "", "scn": null, "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110675, "queue_name": "Queue-5", "thread_name": "Record Sorter", "data": {"scn": "1371666 ", "xid": "1.0x000021.00003467.0190 ", "timestamp": "1699945098 ", "type": "COMMIT }"}, "rsid": "", "scn": "1371666 ", "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110674, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"scn": "1371666 ", "xid": "1.0x000021.00003467.0190 ", "timestamp": "1699945098 ", "type": "START }"}, "rsid": "", "scn": "1371666 ", "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110675, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"rowid": "AAAR7QAAFAAAACGAAk ", "xid": "1.0x000021.00003467.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699945098 ", "pkOrUk": "EMPLOYEE_ID=321 }"}, "rsid": "", "scn": null, "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110676, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"scn": "1371666 ", "xid": "1.0x000021.00003467.0190 ", "timestamp": "1699945098 ", "type": "COMMIT }"}, "rsid": "", "scn": "1371666 ", "xid": "1.0x000021.00003467.0190 "}
{"timestamp": 1699945110777, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "BEGIN", "positionFileID": 1371666, "positionFileOffset": 1, "recordID": 32322, "schemaID": 0, "schemaVersion": 1699945109, "state": "DELIVERED", "threadID": 0, "timestamp": 1699945098, "transactionID": "1.0x000021.00003467.0190", "uniqueNumber": 2689}], "rsid": "", "scn": 1371666, "xid": "1.0x000021.00003467.0190"}
{"timestamp": 1699945110777, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "databaseName": "OBTEST", "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "INSERT", "positionFileID": 1371666, "positionFileOffset": 1, "recordID": 32323, "schema": {"columnEncodings": ["UTF8", "UTF8", "UTF8", "UTF8", "UTF8", "UTF8", "UTF8"], "columnNames": ["EMPLOYEE_ID", "FIRST_NAME", "LAST_NAME", "HIRE_DATE", "JOB_TITLE", "SALARY", "rowid"], "columnNotNulls": "AQAAAAAAAQ==", "columnSchemas": [{"encoding": "UTF8", "indexType": "PRIMARY_KEY", "name": "EMPLOYEE_ID", "notNull": true, "position": 0, "type": "NEWDECIMAL"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "FIRST_NAME", "notNull": false, "position": 1, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "LAST_NAME", "notNull": false, "position": 2, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "HIRE_DATE", "notNull": false, "position": 3, "type": "DATETIME"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "JOB_TITLE", "notNull": false, "position": 4, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "SALARY", "notNull": false, "position": 5, "type": "NEWDECIMAL"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "rowid", "notNull": true, "position": 6, "type": "STRING"}], "columnTypes": "9g8PDA/2/g==", "databaseName": "OBTEST", "encoding": "UTF8", "iD": 0, "instanceInfo": "", "keyRelation": ["(0)"], "nameKey": "OBTEST.T1", "primaryKeyIndexes": [0], "tableName": "T1", "uniqueKeyIndexes": [], "valid": true, "version": 1699868350}, "schemaID": -1, "schemaVersion": -1, "state": "DELIVERED", "tableName": "T1", "threadID": 0, "timestamp": 1699945098, "transactionID": "1.0x000021.00003467.0190", "uniqueNumber": 2690}], "rsid": "", "scn": 1371666, "xid": "1.0x000021.00003467.0190"}
``` 

# 源端+目标端 日志样例: 

```
{"timestamp": 1699953950656, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "609", "ownerId": "null", "tableId": "null", "scn": "1393579", "timestamp": "2023-11-14 09:25:35", "xid": "0A001F00E3060000", "rowid": "AAAR7QAAAAAAAAAAAA", "opType": "START", "segmentType": "UNKNOWN", "segmentName": "null", "redo": "set transaction read write;", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "16", "rollback": "0", "info": "null", "rsId": "0x000022.000002de.0010"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}
{"timestamp": 1699953950660, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "610", "ownerId": "UNKNOWN", "tableId": "OBJ# 73424", "scn": "1393579", "timestamp": "2023-11-14 09:25:35", "xid": "0A001F00E3060000", "rowid": "AAAR7QAAFAAAACGAA8", "opType": "INSERT", "segmentType": "UNKNOWN", "segmentName": "OBJ# 73424", "redo": "insert into \"UNKNOWN\".\"OBJ# 73424\"(\"COL 1\",\"COL 2\",\"COL 3\",\"COL 4\",\"COL 5\",\"COL 6\") values (HEXTORAW(", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "16", "rollback": "0", "info": "Dictionary", "rsId": "0x000022.000002de.0010"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}
{"timestamp": 1699953950667, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "611", "ownerId": "null", "tableId": "null", "scn": "1393580", "timestamp": "2023-11-14 09:25:35", "xid": "0A001F00E3060000", "rowid": "AAAAAAAAAAAAAAAAAA", "opType": "COMMIT", "segmentType": "UNKNOWN", "segmentName": "null", "redo": "commit;", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "400", "rollback": "0", "info": "null", "rsId": "0x000022.000002df.0190"}, "rsid": "0x000022.000002df.0190", "scn": "1393580"}
{"timestamp": 1699953950670, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "33486", "scn": "1393579", "rowid": "null", "timestamp": "1699953935000", "type": "START", "table": "null", "transaction": "0A001F00E3060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}
{"timestamp": 1699953950680, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "33487", "scn": "1393579", "rowid": "AAAR7QAAFAAAACGAA8", "timestamp": "1699953935000", "type": "INSERT", "table": "73424", "transaction": "0A001F00E3060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}
{"timestamp": 1699953950683, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "33488", "scn": "1393580", "rowid": "null", "timestamp": "1699953935000", "type": "COMMIT", "table": "null", "transaction": "0A001F00E3060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002df.0190", "threadid": "1"}, "rsid": "0x000022.000002df.0190", "scn": "1393580"}
{"timestamp": 1699953950689, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "33486", "scn": "1393580", "rowid": "null", "timestamp": "1699953935000", "type": "START", "table": "null", "transaction": "1.0x000022.000002df.0190", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950693, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "33487", "scn": "1393580", "rowid": "AAAR7QAAFAAAACGAA8", "timestamp": "1699953935000", "type": "INSERT", "table": "73424", "transaction": "1.0x000022.000002df.0190", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950700, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "33488", "scn": "1393580", "rowid": "null", "timestamp": "1699953935000", "type": "COMMIT", "table": "null", "transaction": "1.0x000022.000002df.0190", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002df.0190", "threadid": "1"}, "rsid": "0x000022.000002df.0190", "scn": "1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950700, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "33486", "scn": "1393580", "rowid": "null", "timestamp": "1699953935000", "type": "START", "table": "null", "transaction": "1.0x000022.000002df.0190", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950701, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "33487", "scn": "1393580", "rowid": "AAAR7QAAFAAAACGAA8", "timestamp": "1699953935000", "type": "INSERT", "table": "73424", "transaction": "1.0x000022.000002df.0190", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950702, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "33488", "scn": "1393580", "rowid": "null", "timestamp": "1699953935000", "type": "COMMIT", "table": "null", "transaction": "1.0x000022.000002df.0190", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002df.0190", "threadid": "1"}, "rsid": "0x000022.000002df.0190", "scn": "1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950714, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"scn": "1393580 ", "xid": "1.0x000022.000002df.0190 ", "timestamp": "1699953935 ", "type": "START }"}, "scn": "1393580 ", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950719, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"rowid": "AAAR7QAAFAAAACGAA8 ", "xid": "1.0x000022.000002df.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699953935 ", "pkOrUk": "EMPLOYEE_ID=346 }"}, "scn": "", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950720, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"scn": "1393580 ", "xid": "1.0x000022.000002df.0190 ", "timestamp": "1699953935 ", "type": "COMMIT }"}, "scn": "1393580 ", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950723, "queue_name": "Queue-5", "thread_name": "Record Sorter", "data": {"scn": "1393580 ", "xid": "1.0x000022.000002df.0190 ", "timestamp": "1699953935 ", "type": "START }"}, "scn": "1393580 ", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950726, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"scn": "1393580 ", "xid": "1.0x000022.000002df.0190 ", "timestamp": "1699953935 ", "type": "START }"}, "scn": "1393580 ", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950728, "queue_name": "Queue-5", "thread_name": "Record Sorter", "data": {"rowid": "AAAR7QAAFAAAACGAA8 ", "xid": "1.0x000022.000002df.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699953935 ", "pkOrUk": "EMPLOYEE_ID=346 }"}, "scn": "", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950731, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"rowid": "AAAR7QAAFAAAACGAA8 ", "xid": "1.0x000022.000002df.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699953935 ", "pkOrUk": "EMPLOYEE_ID=346 }"}, "scn": "", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950733, "queue_name": "Queue-5", "thread_name": "Record Sorter", "data": {"scn": "1393580 ", "xid": "1.0x000022.000002df.0190 ", "timestamp": "1699953935 ", "type": "COMMIT }"}, "scn": "1393580 ", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950736, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"scn": "1393580 ", "xid": "1.0x000022.000002df.0190 ", "timestamp": "1699953935 ", "type": "COMMIT }"}, "scn": "1393580 ", "xid": "1.0x000022.000002df.0190 "}
{"timestamp": 1699953950843, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "BEGIN", "positionFileID": 1393580, "positionFileOffset": 1, "recordID": 33454, "schemaID": 0, "schemaVersion": 1699953950, "state": "DELIVERED", "threadID": 0, "timestamp": 1699953935, "transactionID": "1.0x000022.000002df.0190", "uniqueNumber": 2810}], "scn": 1393580, "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950849, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "databaseName": "OBTEST", "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "INSERT", "positionFileID": 1393580, "positionFileOffset": 1, "recordID": 33455, "schema": {"columnEncodings": ["UTF8", "UTF8", "UTF8", "UTF8", "UTF8", "UTF8", "UTF8"], "columnNames": ["EMPLOYEE_ID", "FIRST_NAME", "LAST_NAME", "HIRE_DATE", "JOB_TITLE", "SALARY", "rowid"], "columnNotNulls": "AQAAAAAAAQ==", "columnSchemas": [{"encoding": "UTF8", "indexType": "PRIMARY_KEY", "name": "EMPLOYEE_ID", "notNull": true, "position": 0, "type": "NEWDECIMAL"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "FIRST_NAME", "notNull": false, "position": 1, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "LAST_NAME", "notNull": false, "position": 2, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "HIRE_DATE", "notNull": false, "position": 3, "type": "DATETIME"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "JOB_TITLE", "notNull": false, "position": 4, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "SALARY", "notNull": false, "position": 5, "type": "NEWDECIMAL"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "rowid", "notNull": true, "position": 6, "type": "STRING"}], "columnTypes": "9g8PDA/2/g==", "databaseName": "OBTEST", "encoding": "UTF8", "iD": 0, "instanceInfo": "", "keyRelation": ["(0)"], "nameKey": "OBTEST.T1", "primaryKeyIndexes": [0], "tableName": "T1", "uniqueKeyIndexes": [], "valid": true, "version": 1699868350}, "schemaID": -1, "schemaVersion": -1, "state": "DELIVERED", "tableName": "T1", "threadID": 0, "timestamp": 1699953935, "transactionID": "1.0x000022.000002df.0190", "uniqueNumber": 2811}], "scn": 1393580, "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950851, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "COMMIT", "positionFileID": 1393580, "positionFileOffset": 1, "recordID": 33456, "schemaID": 0, "schemaVersion": 1699953950, "state": "DELIVERED", "threadID": 0, "timestamp": 1699953935, "transactionID": "1.0x000022.000002df.0190", "uniqueNumber": 2812}], "scn": 1393580, "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950866, "queue_name": "(From store)", "thread_name": "DRC-Client-Thread", "data": {"recordType": "BEGIN", "uniqueId": "1.0x000022.000002df.0190", "scn": "1@1393580", "sourceDBName": null, "sourceTableName": null}, "scn": "1@1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950871, "queue_name": "(From store)", "thread_name": "DRC-Client-Thread", "data": {"recordType": "INSERT", "uniqueId": null, "scn": "1@1393580", "sourceDBName": "OBTEST", "sourceTableName": "T1"}, "scn": "1@1393580", "xid": null}
{"timestamp": 1699953950873, "queue_name": "(From store)", "thread_name": "DRC-Client-Thread", "data": {"recordType": "COMMIT", "uniqueId": "1.0x000022.000002df.0190", "scn": "1@1393580", "sourceDBName": null, "sourceTableName": null}, "scn": "1@1393580", "xid": "1.0x000022.000002df.0190"}
{"timestamp": 1699953950876, "queue_name": "Queue-11", "thread_name": "DRC-Client-Thread", "data": {"batchType": "DML", "transactionId": "1@1393580", "dataSize": 584, "recordSize": 1}, "scn": "1@1393580"}
{"timestamp": 1699953950878, "queue_name": "Queue-12", "thread_name": "forward_slot0-(ETLProcessor)-queue_slot1", "data": {"batchType": "DML", "transactionId": "1@1393580", "dataSize": 584, "recordSize": 1}, "scn": "1@1393580"}
{"timestamp": 1699953950893, "queue_name": "(before JDBC)", "thread_name": "sinkTask-0", "data": {"batchType": "DML", "transactionId": "1@1393580", "dataSize": 584, "recordSize": 1}, "scn": "1@1393580"}
{"timestamp": 1699953950893, "queue_name": "Queue-13", "thread_name": "queue_slot1-()-TransactionScheduler", "data": {"batchType": "DML", "transactionId": "1@1393580", "dataSize": 584, "recordSize": 1}, "scn": "1@1393580"}
{"timestamp": 1699953950914, "queue_name": "(after JDBC)", "thread_name": "sinkTask-0", "data": {"batchType": "DML", "transactionId": "1@1393580", "dataSize": 584, "recordSize": 1}, "scn": "1@1393580"}

``` 

不包括SCN 1393580的行包括: 

```
{"timestamp": 1699953950656, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "609", "ownerId": "null", "tableId": "null", "scn": "1393579", "timestamp": "2023-11-14 09:25:35", "xid": "0A001F00E3060000", "rowid": "AAAR7QAAAAAAAAAAAA", "opType": "START", "segmentType": "UNKNOWN", "segmentName": "null", "redo": "set transaction read write;", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "16", "rollback": "0", "info": "null", "rsId": "0x000022.000002de.0010"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}
{"timestamp": 1699953950660, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "610", "ownerId": "UNKNOWN", "tableId": "OBJ# 73424", "scn": "1393579", "timestamp": "2023-11-14 09:25:35", "xid": "0A001F00E3060000", "rowid": "AAAR7QAAFAAAACGAA8", "opType": "INSERT", "segmentType": "UNKNOWN", "segmentName": "OBJ# 73424", "redo": "insert into \"UNKNOWN\".\"OBJ# 73424\"(\"COL 1\",\"COL 2\",\"COL 3\",\"COL 4\",\"COL 5\",\"COL 6\") values (HEXTORAW(", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "16", "rollback": "0", "info": "Dictionary", "rsId": "0x000022.000002de.0010"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}

{"timestamp": 1699953950670, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "33486", "scn": "1393579", "rowid": "null", "timestamp": "1699953935000", "type": "START", "table": "null", "transaction": "0A001F00E3060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}
{"timestamp": 1699953950680, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "33487", "scn": "1393579", "rowid": "AAAR7QAAFAAAACGAA8", "timestamp": "1699953935000", "type": "INSERT", "table": "73424", "transaction": "0A001F00E3060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.000002de.0010", "threadid": "1"}, "rsid": "0x000022.000002de.0010", "scn": "1393579"}

{"timestamp": 1699953950719, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"rowid": "AAAR7QAAFAAAACGAA8 ", "xid": "1.0x000022.000002df.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699953935 ", "pkOrUk": "EMPLOYEE_ID=346 }"}, "scn": "", "xid": "1.0x000022.000002df.0190 "}

{"timestamp": 1699953950728, "queue_name": "Queue-5", "thread_name": "Record Sorter", "data": {"rowid": "AAAR7QAAFAAAACGAA8 ", "xid": "1.0x000022.000002df.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699953935 ", "pkOrUk": "EMPLOYEE_ID=346 }"}, "scn": "", "xid": "1.0x000022.000002df.0190 "}

{"timestamp": 1699953950731, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"rowid": "AAAR7QAAFAAAACGAA8 ", "xid": "1.0x000022.000002df.0190 ", "table": "OBTEST.T1 ", "type": "INSERT ", "timestamp": "1699953935 ", "pkOrUk": "EMPLOYEE_ID=346 }"}, "scn": "", "xid": "1.0x000022.000002df.0190 "}

``` 

其中: 

  - 队列1/2属于正常现象, commit的 SCN回填, 是在Queue-3完成. 队列1/2的SCN仅代表记录的SCN, 而非事务的SCN
  - 队列 5/6/8 是脚本逻辑错误, 进行修正

修正后的结果: 

```
{"timestamp": 1699977112858, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "29027", "ownerId": "null", "tableId": "null", "scn": "1450187", "timestamp": "2023-11-14 15:51:32", "xid": "0A001700FE060000", "rowid": "AAAR7QAAAAAAAAAAAA", "opType": "START", "segmentType": "UNKNOWN", "segmentName": "null", "redo": "set transaction read write;", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "16", "rollback": "0", "info": "null", "rsId": "0x000022.00003c9b.0010"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450187"}
{"timestamp": 1699977112860, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "29028", "ownerId": "UNKNOWN", "tableId": "OBJ# 73424", "scn": "1450187", "timestamp": "2023-11-14 15:51:32", "xid": "0A001700FE060000", "rowid": "AAAR7QAAFAAAACDAAF", "opType": "INSERT", "segmentType": "UNKNOWN", "segmentName": "OBJ# 73424", "redo": "insert into \"UNKNOWN\".\"OBJ# 73424\"(\"COL 1\",\"COL 2\",\"COL 3\",\"COL 4\",\"COL 5\",\"COL 6\") values (HEXTORAW(", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "16", "rollback": "0", "info": "Dictionary", "rsId": "0x000022.00003c9b.0010"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450187"}
{"timestamp": 1699977112865, "queue_name": "Queue-1", "thread_name": "LogEntryPrimaryFetcher#1", "data": {"rownum": "29029", "ownerId": "null", "tableId": "null", "scn": "1450188", "timestamp": "2023-11-14 15:51:32", "xid": "0A001700FE060000", "rowid": "AAAAAAAAAAAAAAAAAA", "opType": "COMMIT", "segmentType": "UNKNOWN", "segmentName": "null", "redo": "commit;", "csf": "0", "threadId": "1", "rbaLogFileSeq": "34", "rbaBlockId": "1", "rbaByteOffset": "352", "rollback": "0", "info": "null", "rsId": "0x000022.00003c9c.0160"}, "rsid": "0x000022.00003c9c.0160", "scn": "1450188"}
{"timestamp": 1699977112869, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "35659", "scn": "1450187", "rowid": "null", "timestamp": "1699977092000", "type": "START", "table": "null", "transaction": "0A001700FE060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9b.0010", "threadid": "1"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450187"}
{"timestamp": 1699977112880, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "35660", "scn": "1450187", "rowid": "AAAR7QAAFAAAACDAAF", "timestamp": "1699977092000", "type": "INSERT", "table": "73424", "transaction": "0A001700FE060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9b.0010", "threadid": "1"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450187"}
{"timestamp": 1699977112882, "queue_name": "Queue-2", "thread_name": "Oracle Log Extractor Instance#1", "data": {"id": "35661", "scn": "1450188", "rowid": "null", "timestamp": "1699977092000", "type": "COMMIT", "table": "null", "transaction": "0A001700FE060000", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9c.0160", "threadid": "1"}, "rsid": "0x000022.00003c9c.0160", "scn": "1450188"}
{"timestamp": 1699977112886, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "35659", "scn": "1450188", "rowid": "null", "timestamp": "1699977092000", "type": "START", "table": "null", "transaction": "1.0x000022.00003c9c.0160", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9b.0010", "threadid": "1"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112891, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "35660", "scn": "1450188", "rowid": "AAAR7QAAFAAAACDAAF", "timestamp": "1699977092000", "type": "INSERT", "table": "73424", "transaction": "1.0x000022.00003c9c.0160", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9b.0010", "threadid": "1"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112896, "queue_name": "Queue-3", "thread_name": "Oracle LogRecord Serializer Instance#1", "data": {"id": "35661", "scn": "1450188", "rowid": "null", "timestamp": "1699977092000", "type": "COMMIT", "table": "null", "transaction": "1.0x000022.00003c9c.0160", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9c.0160", "threadid": "1"}, "rsid": "0x000022.00003c9c.0160", "scn": "1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112898, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "35659", "scn": "1450188", "rowid": "null", "timestamp": "1699977092000", "type": "START", "table": "null", "transaction": "1.0x000022.00003c9c.0160", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9b.0010", "threadid": "1"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112900, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "35660", "scn": "1450188", "rowid": "AAAR7QAAFAAAACDAAF", "timestamp": "1699977092000", "type": "INSERT", "table": "73424", "transaction": "1.0x000022.00003c9c.0160", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9b.0010", "threadid": "1"}, "rsid": "0x000022.00003c9b.0010", "scn": "1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112903, "queue_name": "Queue-4", "thread_name": "Oracle LogRecord Converter Prepare", "data": {"id": "35661", "scn": "1450188", "rowid": "null", "timestamp": "1699977092000", "type": "COMMIT", "table": "null", "transaction": "1.0x000022.00003c9c.0160", "fileid": "34", "blockid": "1", "rollback": "0", "rsid": "0x000022.00003c9c.0160", "threadid": "1"}, "rsid": "0x000022.00003c9c.0160", "scn": "1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112915, "queue_name": "Queue-5", "thread_name": "Oracle LogRecord Converter Core", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "START", "schema": null, "table": null, "pk": ""}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112924, "queue_name": "Queue-5", "thread_name": "Oracle LogRecord Converter Core", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "INSERT", "schema": "OBTEST", "table": "T1", "pk": "EMPLOYEE_ID=505"}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112924, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "START", "schema": null, "table": null, "pk": ""}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112932, "queue_name": "Queue-5", "thread_name": "Oracle LogRecord Converter Core", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "COMMIT", "schema": null, "table": null, "pk": ""}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112934, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "INSERT", "schema": "OBTEST", "table": "T1", "pk": "EMPLOYEE_ID=505"}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112943, "queue_name": "Queue-6 (No Look-up)", "thread_name": "Record Dispatcher", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "COMMIT", "schema": null, "table": null, "pk": ""}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112949, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "START", "schema": null, "table": null, "pk": ""}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112955, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "INSERT", "schema": "OBTEST", "table": "T1", "pk": "EMPLOYEE_ID=505"}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977112959, "queue_name": "Queue-8", "thread_name": "Record Sorter", "data": {"transactionId": "1.0x000022.00003c9c.0160", "scn": 1450188, "type": "COMMIT", "schema": null, "table": null, "pk": ""}, "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977113068, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "BEGIN", "positionFileID": 1450188, "positionFileOffset": 1, "recordID": 35627, "schemaID": 0, "schemaVersion": 1699977112, "state": "DELIVERED", "threadID": 0, "timestamp": 1699977092, "transactionID": "1.0x000022.00003c9c.0160", "uniqueNumber": 2969}], "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977113077, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "databaseName": "OBTEST", "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "INSERT", "positionFileID": 1450188, "positionFileOffset": 1, "recordID": 35628, "schema": {"columnEncodings": ["UTF8", "UTF8", "UTF8", "UTF8", "UTF8", "UTF8", "UTF8"], "columnNames": ["EMPLOYEE_ID", "FIRST_NAME", "LAST_NAME", "HIRE_DATE", "JOB_TITLE", "SALARY", "rowid"], "columnNotNulls": "AQAAAAAAAQ==", "columnSchemas": [{"encoding": "UTF8", "indexType": "PRIMARY_KEY", "name": "EMPLOYEE_ID", "notNull": true, "position": 0, "type": "NEWDECIMAL"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "FIRST_NAME", "notNull": false, "position": 1, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "LAST_NAME", "notNull": false, "position": 2, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "HIRE_DATE", "notNull": false, "position": 3, "type": "DATETIME"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "JOB_TITLE", "notNull": false, "position": 4, "type": "VARCHAR"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "SALARY", "notNull": false, "position": 5, "type": "NEWDECIMAL"}, {"encoding": "UTF8", "indexType": "NORMAL", "name": "rowid", "notNull": true, "position": 6, "type": "STRING"}], "columnTypes": "9g8PDA/2/g==", "databaseName": "OBTEST", "encoding": "UTF8", "iD": 0, "instanceInfo": "", "keyRelation": ["(0)"], "nameKey": "OBTEST.T1", "primaryKeyIndexes": [0], "tableName": "T1", "uniqueKeyIndexes": [], "valid": true, "version": 1699868350}, "schemaID": -1, "schemaVersion": -1, "state": "DELIVERED", "tableName": "T1", "threadID": 0, "timestamp": 1699977092, "transactionID": "1.0x000022.00003c9c.0160", "uniqueNumber": 2970}], "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977113078, "queue_name": "Queue-9 (Store)", "thread_name": "pool-2-thread-1", "data": [{"bytes": {"contiguous": true, "direct": false, "readOnly": false, "readable": true, "writable": true}, "dbType": 3, "future": {"cancelled": false, "done": true}, "messageType": "COMMIT", "positionFileID": 1450188, "positionFileOffset": 1, "recordID": 35629, "schemaID": 0, "schemaVersion": 1699977112, "state": "DELIVERED", "threadID": 0, "timestamp": 1699977092, "transactionID": "1.0x000022.00003c9c.0160", "uniqueNumber": 2971}], "scn": 1450188, "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977113095, "queue_name": "(From store)", "thread_name": "DRC-Client-Thread", "data": {"recordType": "BEGIN", "uniqueId": "1.0x000022.00003c9c.0160", "scn": "1@1450188", "sourceDBName": null, "sourceTableName": null}, "scn": "1@1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977113103, "queue_name": "(From store)", "thread_name": "DRC-Client-Thread", "data": {"recordType": "INSERT", "uniqueId": null, "scn": "1@1450188", "sourceDBName": "OBTEST", "sourceTableName": "T1"}, "scn": "1@1450188", "xid": null}
{"timestamp": 1699977113105, "queue_name": "(From store)", "thread_name": "DRC-Client-Thread", "data": {"recordType": "COMMIT", "uniqueId": "1.0x000022.00003c9c.0160", "scn": "1@1450188", "sourceDBName": null, "sourceTableName": null}, "scn": "1@1450188", "xid": "1.0x000022.00003c9c.0160"}
{"timestamp": 1699977113107, "queue_name": "Queue-11", "thread_name": "DRC-Client-Thread", "data": {"batchType": "DML", "transactionId": "1@1450188", "dataSize": 584, "recordSize": 1}, "scn": "1@1450188"}
{"timestamp": 1699977113110, "queue_name": "Queue-12", "thread_name": "forward_slot0-(ETLProcessor)-queue_slot1", "data": {"batchType": "DML", "transactionId": "1@1450188", "dataSize": 584, "recordSize": 1}, "scn": "1@1450188"}
{"timestamp": 1699977113124, "queue_name": "Queue-13", "thread_name": "queue_slot1-()-TransactionScheduler", "data": {"batchType": "DML", "transactionId": "1@1450188", "dataSize": 584, "recordSize": 1}, "scn": "1@1450188"}
{"timestamp": 1699977113126, "queue_name": "(before JDBC)", "thread_name": "sinkTask-45", "data": {"batchType": "DML", "transactionId": "1@1450188", "dataSize": 584, "recordSize": 1}, "scn": "1@1450188"}
{"timestamp": 1699977113151, "queue_name": "(after JDBC)", "thread_name": "sinkTask-45", "data": {"batchType": "DML", "transactionId": "1@1450188", "dataSize": 584, "recordSize": 1}, "scn": "1@1450188"}

```
