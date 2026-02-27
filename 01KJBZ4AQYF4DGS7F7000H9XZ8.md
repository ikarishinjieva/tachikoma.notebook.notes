---
title: 20231204 - 整理OMS的全量复制链路
confluence_page_id: 2589822
created_at: 2023-12-04T03:35:26+00:00
updated_at: 2023-12-13T05:47:56+00:00
---

# 目录

# 处理器1 - 扫描表的元数据, 获取有多少张表以及表的结构

  - 输入: 源库的表信息
    - 堆栈 

```
com.oceanbase.oms.connector.coordinator.BootStrapPanel#boot
	-> com.oceanbase.oms.connector.coordinator.BootStrapPanel#initConditions (初始化表名的搜索条件)
	-> com.oceanbase.oms.connector.source.dataflow.DataFlowSourceBooster#initSourceEnv
		-> com.oceanbase.oms.connector.source.dataflow.DataFlowSourceBooster#initTables (获取 表的元数据)
		-> com.oceanbase.oms.connector.source.dataflow.DataFlowSliceProvider#init
			-> com.oceanbase.oms.connector.source.dataflow.SliceTaskManager#filterNeedPullTableInfos (读取表的checkpoint)
			-> com.oceanbase.oms.connector.source.dataflow.SliceTaskManager#init (输出到下游)
``` 
  - 线程名: main
  - 输出队列: com.oceanbase.oms.connector.source.dataflow.SliceTaskManager#sliceTaskQueue
    - Arthas命令: 

```
[arthas@52976]$ vmtool --action getInstances --className com.oceanbase.oms.connector.source.dataflow.SliceTaskManager --limit 100 --express 'instances[0].sliceTaskQueue'
@LinkedBlockingDeque[isEmpty=true;size=0]
``` 

# 处理器2 - 数据切分器, 获取对数据进行切分的边界

  - 输入队列: com.oceanbase.oms.connector.source.dataflow.SliceTaskManager#sliceTaskQueue
  - 线程名: slice-worker-n (多个线程)
    - 管理方: SliceTaskManager
    - 不同表的扫描分配给不同的线程
  - 工作代码:

```
com.oceanbase.oms.connector.source.dataflow.SliceTaskManager.InnerSliceTask#run
	-> com.oceanbase.oms.connector.source.dataflow.DataFlowSliceTask#doSlice
		-> com.oceanbase.oms.dataflow.slice.LocalSliceProvider#provider 
			-> com.oceanbase.oms.dataflow.slice.BlockSliceService#slice (使用Block方式 切片)
				-> com.oceanbase.oms.connector.source.dataflow.DataFlowSliceTask#handleNewSlice (处理新的切片)
					-> 将切片添加到checkpoint中
					-> 输出到队列
``` 
  - 可选的切分方式: All / PK / PKIncrement / Block
  - 输出队列: com.oceanbase.oms.connector.source.dataflow.DataFlowSliceProvider#sliceQueue 
    - Arthas命令: 

```
[arthas@3690]$ vmtool --action getInstances --className com.oceanbase.oms.connector.source.dataflow.DataFlowSliceProvider --limit 100 --express 'instances[0].sliceQueue'
@ArrayBlockingQueue[
    @SliceQueryConditions[clientId: {OBTEST.STOCK BLOCK 97 [OMS_OBJECT_NUMBER:76329,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:24064,OMS_ROW_NUMBER:0] - [OMS_OBJECT_NUMBER:76329,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:24192,OMS_ROW_NUMBER:0]}; finished: false],
    @SliceQueryConditions[clientId: {OBTEST.ORDER_LINE BLOCK 25 [OMS_OBJECT_NUMBER:76327,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:93440,OMS_ROW_NUMBER:0] - [OMS_OBJECT_NUMBER:76327,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:93568,OMS_ROW_NUMBER:0]}; finished: false],
  
...
``` 
  - 特殊逻辑:
    - 设置 优化器版本: alter session set OPTIMIZER_FEATURES_ENABLE= ?
    - 判断表 是否是 bigfile tablespace: SELECT RELATIVE_FNO FROM DBA_EXTENTS WHERE OWNER='{}' AND SEGMENT_NAME='{}' AND ROWNUM<2, 
    1.        1. 判断RELATIVE_FNO是否为1024 (不明确其中的逻辑关系), 如果是1024, 则为bigfile table
       2. 对于起始的块号, 需要转换成bigfile的块号 (com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient#smallFileFnoBlockNumberToBigfile )
    - 查询块号:
      - SQL (如果是分区表, 还会增加partition的相关条件) (com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient#createBlockQuerySql)
      - 样例: 

```
SELECT  OMS_OBJECT_NUMBER, OMS_RELATIVE_FNO, OMS_BLOCK_NUMBER, OMS_ROW_NUMBER, BLOCKS FROM ( SELECT OBJ.DATA_OBJECT_ID OMS_OBJECT_NUMBER,EXT.RELATIVE_FNO OMS_RELATIVE_FNO, EXT.BLOCK_ID OMS_BLOCK_NUMBER, 0 OMS_ROW_NUMBER , EXT.BLOCKS BLOCKS   FROM DBA_EXTENTS EXT, ALL_OBJECTS OBJ WHERE EXT.OWNER = 'OBTEST' AND EXT.OWNER = OBJ.OWNER AND EXT.SEGMENT_NAME = 'ORDER_LINE'
AND EXT.SEGMENT_TYPE IN ('TABLE', 'TABLE PARTITION','TABLE SUBPARTITION') AND EXT.SEGMENT_NAME = OBJ.OBJECT_NAME   ) ORDER BY OMS_OBJECT_NUMBER, OMS_RELATIVE_FNO, OMS_BLOCK_NUMBER

---
输出样例:

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
        76327        5        19552      0      8
        76327        5        67984      0      8
        76327        5        67992      0      8
        76327        5        68008      0      8
        76327        5        68016      0      8
        76327        5        68024      0      8
        76327        5        68040      0      8
        76327        5        68048      0      8
        76327        5        68056      0      8
        76327        5        68072      0      8
        76327        5        68080      0      8
``` 
    - 进行切分: OMS_BLOCK_NUMBER 为起始地址, BLOCKS为数量, 切分大小为 配置limitator.splitor.blocks

# 处理器3: 获取数据

  - 输入队列: com.oceanbase.oms.connector.source.dataflow.DataFlowSliceProvider#sliceQueue
  - 线程名: 
    - sourceTask-n (多个线程)
    - 管理方: SourceTaskManager
  - 工作代码: 

```
com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask#run
	-> com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask#pollQueryConditionWithRetry
		-> com.oceanbase.oms.connector.source.dataflow.DataFlowSource#poll (通过JDBC获取数据)
		-> com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask#handleRecordBatch (处理数据)
			-> com.oceanbase.oms.connector.common.model.BatchProcessor#process (输出给 ETLBatchProcessor 处理)
			-> com.oceanbase.oms.connector.common.model.IOffer#offer (将处理的结果, 输出给下游队列)
``` 
  - 输出队列: com.oceanbase.connector.framework.threadmanager.sourcetask.SourceTaskManager#sink
  - 特殊逻辑: 
    - 对于有一致性的表, 使用 jdbcClient.queryDataBySliceConsistency
      - 仅对MySQL有效, 设置隔离级别为RR, 对表上读锁, 然后下SQL (com.oceanbase.oms.dataflow.jdbcclient.mysqlbased.mysql.MysqlQueryClient#queryDataBySliceConsistency)
    - 否则, 使用 jdbcClient.queryDataBySlice
    - SQL格式: 

```
"SELECT /*+ ROWID(\"%s\") */ %s FROM %s %s WHERE ROWID>? AND ROWID<=?"
``` 
    - 使用 转换函数, 将SQL结果转成对象 (com.oceanbase.oms.connector.source.dataflow.RecordValueConvertFunction)
    - 构建一个 StreamRecordBatchBuilder, 将 fetcher和转换函数 组织在一起, 包装成一个 数据行的迭代器
    - 限流入口: 堆栈: 

```
ts=2023-12-03 11:59:56;thread_name=sourceTask-7;id=43;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.oms.connector.common.ThrottleCenter.inThrottle()
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.handleRecordBatch(ConditionedSourceConnectorTask.java:183)
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.pollQueryConditionWithRetry(ConditionedSourceConnectorTask.java:125)
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.run(ConditionedSourceConnectorTask.java:68)
        at java.lang.Thread.run(Thread.java:853)
``` 

# 处理器4: 对分片表进行分发

  - 输入队列: com.oceanbase.connector.framework.threadmanager.sourcetask.SourceTaskManager#sink
  - 处理器: com.oceanbase.oms.connector.dispatcher.shard.DynamicShardDispatcher 

```
com.oceanbase.oms.connector.dispatcher.shard.DynamicShardDispatcher#innerSubmitDmlBatch
-> com.oceanbase.oms.connector.dispatcher.shard.DynamicShardDispatcher#tryDispatchAffectRecordBatch
	-> com.oceanbase.oms.connector.dispatcher.shard.ShardBucket#drainRecordBatch (输出到下游队列)
``` 
  - 线程名: queue_slot1-()-null
  - 输出队列: com.oceanbase.oms.connector.dispatcher.shard.DynamicShardDispatcher#readyBatch

  

# 处理器5: 回放数据

  - 输入队列: com.oceanbase.oms.connector.dispatcher.shard.DynamicShardDispatcher (包装成队列, 实际访问readyBatch)
  - 处理器: SyncSinkConnectorTask 

```
com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask#run
	-> source.poll() 
	-> OBOracleJDBCSink#offer
		-> com.oceanbase.oms.connector.jdbc.sink.DefaultJDBCSink#offer
			-> com.oceanbase.oms.connector.jdbc.sink.Writer#flushRecords (回放数据)
``` 

  

  - 线程名: sinkTask-n (多个线程)
  - 特殊机制:
    - 水位监视器: 由OBOracleJDBCSinkBooster#startOBHighWatermarkMonitor启动, 监控目标数据库的 CPU和内存使用的水位 (通过SQL), 如果水位超过阈值, 记录日志

    - 限流出口: 堆栈: 

```
ts=2023-12-03 12:00:08;thread_name=sinkTask-2;id=35;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.oms.connector.common.ThrottleCenter.outThrottle()
        at com.oceanbase.oms.connector.dispatcher.shard.AbstractShardDispatcher.releaseShardRecordBatch(AbstractShardDispatcher.java:204)
        at com.oceanbase.oms.connector.dispatcher.shard.AbstractShardDispatcher.releaseShardRecordBatchWithStat(AbstractShardDispatcher.java:209)
        at com.oceanbase.oms.connector.dispatcher.shard.ShardRecordBatch.onBatchStateChange(ShardRecordBatch.java:126)
        at com.oceanbase.oms.connector.common.util.RecordBatchUtil.invokeBatchCallBack(RecordBatchUtil.java:14)
        at com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask.run(SyncSinkConnectorTask.java:49)
        at java.lang.Thread.run(Thread.java:853)
 
ts=2023-12-03 12:00:08;thread_name=sinkTask-2;id=35;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.oms.connector.common.ThrottleCenter.outThrottle()
        at com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask.run(SyncSinkConnectorTask.java:51)
        at java.lang.Thread.run(Thread.java:853)
``` 

# 回执6:

  - 堆栈: 

```
com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask#run
	-> com.oceanbase.oms.connector.common.util.RecordBatchUtil#invokeBatchCallBack
		-> com.oceanbase.oms.connector.source.dataflow.AckBatchListener#onNewBatchState (处理回执)
			-> com.oceanbase.oms.connector.source.dataflow.CheckpointStore#ackCheckpoint (更新检查点, 将已确认的分片标记为完成)
``` 

# 特殊机制

各机制的运作阶段, 可以在上述流程中获取

  - 检查点机制 (断点续传)
    - 记录各表拆分的分片, 以及哪些分片已经完成
    - checkpoint文件样例: 

```
[root@R740-26 10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002]# pwd
/u01/ds/run/10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002
[root@R740-26 10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002]# grep '' oms.connector.checkpoint
{"checkpoint":"{}","version":"3.4.0","port":16001,"gmt":1701161062,"status":"done","firstRoundStartingGmtTime":1701160659,"startingGmtTime":1701160665,"finishedTables":9,"totalTables":9,"failedTables":0,"accumulatedRunningTime":397,"predictedTimeToFinish":0,"progress":"1.0","recordProgress":"1.0","processedRecords":5089999,"capacity":5089999,"inconsistentQuantity":0,"consistentQuantity":0,"numberOfInsertedRecords":5089999,"numberOfDeletedRecords":0,"numberOfUpdatedRecords":0}
[root@R740-26 10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002]#
``` 
  - 水位监视器
    - 监视目标数据库的资源消耗, 超过阈值以后, 记录日志 (但没有具体动作)
  - 限流机制
    - 流量定义: 从 获取了源端的数据 到 数据回放到目标端 这一段时间的流量
    - 限流: 可以按照以下规则进行流量限制:
      - 内存限制
      - 每秒 数据行数
      - 每秒 字节数
      - 总数据行数

以下机制是周期性机制:

  - 心跳机制: 向文件系统维护 心跳文件 (HeartBeatManager)
    - 样例: 

```
[root@R740-26 1]# pwd
/u01/ds/run/10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002/migrate/1
[root@R740-26 1]# grep '' heartbeat
{"accumulatedRunningTime":397,"capacity":5089999,"consistentQuantity":0,"dstRps":10313,"dstRpsRef":8000,"dstRt":0,"dstRtRef":1,"dstSpeed":7643410,"failedTables":0,"finishedTables":9,"firstRoundStartingGmtTime":1701160659,"id":"1","inconsistentQuantity":0,"message":"","numberOfDeletedRecords":0,"numberOfInsertedRecords":5089999,"numberOfUpdatedRecords":0,"predictedTimeToFinish":0,"processedRecords":5089999,"progress":"1.0","recordProgress":"1.0","reportingGmtTime":1701161056,"rps":10313,"srcRps":10313,"srcRpsRef":8000,"srcRt":0,"srcRtRef":1,"srcSpeed":7643410,"srcSpeedRef":8388608,"startingGmtTime":1701160659,"status":"done","subId":"1","totalTables":9,"type":"migrate"}
``` 
  - 任务状态文件: 向文件系统维护 任务的状态文件 (OverviewManager)
    - 样例: 

```
[root@R740-26 1]# grep '' overview*
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701160672,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":7,"capacity":100000,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"ITEM","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":0,"inserted":100000,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":0,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"ITEM","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701160682,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":17,"capacity":300000,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"HISTORY","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":1682,"inserted":300000,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":1682,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"HISTORY","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701160701,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":36,"capacity":10,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"WAREHOUSE","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":0,"inserted":10,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":0,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"WAREHOUSE","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701160710,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":45,"capacity":100,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"DISTRICT","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":4,"inserted":100,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":4,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"DISTRICT","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701160731,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":66,"capacity":90000,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"NEW_ORDER","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":0,"inserted":90000,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":0,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"NEW_ORDER","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701160746,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":81,"capacity":300000,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"ORDERS","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":9638,"inserted":300000,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":9638,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"ORDERS","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701161019,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":354,"capacity":300000,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"CUSTOMER","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":3,"inserted":300000,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":3,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"CUSTOMER","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701161049,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":384,"capacity":2999889,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"ORDER_LINE","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":4438,"inserted":2999889,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":4438,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"ORDER_LINE","updated":0}],"type":"migrate"}
overview-done:{"empty":false,"id":"np_59s1c3qy1kkg","name":"overview-done","reportingGmtTime":1701161056,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[{"accumulatedRunningTime":391,"capacity":1000000,"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","consistent":0,"deleted":0,"destSchema":"OBTEST","destTable":"STOCK","firstRoundStartingTime":1701160665,"imageOnly":0,"insertTps":8841,"inserted":1000000,"masterOnly":0,"mismatched":0,"progress":"1.0","readDestTps":0,"readTps":8841,"schema":"OBTEST","startingDate":"2023-11-28 16:37:45","status":"Migrate finished","table":"STOCK","updated":0}],"type":"migrate"}
overview-running:{"empty":true,"id":"np_59s1c3qy1kkg","name":"overview-running","reportingGmtTime":1701161056,"startingGmtTime":1701160665,"subId":"1","tableOverviews":[],"type":"migrate"}
``` 

# 日志梳理

通过vim, 编辑/u01/ds/plugins/jdbc_connector/jdbc_connector.jar中的logback.xml文件

将日志格式中, 增加%c, 打印class名称, 方便阅读. 

日志位置举例: /u01/ds/run/10.186.16.126-9000:connector_v2:np_59zi2w5dahgg-full_trans-1-0:0009000003/logs/connector.log

日志样例: [connector.log.zip]

以一行日志为例: 

```
[2023-11-30 18:50:15.147] [com.oceanbase.oms.connector.extend.Preprocessor] [INFO] [main] [auto set coordinator config: {bridgeQueueSize=128, throttleMemoryBound=2147483648} by metrics LARGE]
``` 

  - main为线程名
  - com.oceanbase.oms.connector.extend.Preprocessor为打印该行日志的类名

流程关键日志举例: 

```
...
调整配置: 
[2023-11-30 18:50:15.147] [com.oceanbase.oms.connector.extend.Preprocessor] [INFO] [main] [auto set coordinator config: {bridgeQueueSize=128, throttleMemoryBound=2147483648} by metrics LARGE]
[2023-11-30 18:50:15.153] [com.oceanbase.oms.connector.extend.Preprocessor] [INFO] [main] [auto set source config: {sourceBatchMemorySize=16777216, batchSize=512} by metrics LARGE]
...
 
建立源端连接: 
[2023-11-30 18:50:15.797] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [init druid dataSource with loginTimeout[300], maxWaitMillis[1200000], maxActive[50], initActive[0], minIdle[0]]
[2023-11-30 18:50:15.800] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [connect with url [jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER?rewriteBatchedStatements=true&GetConnectionTimeout=20000&socketTimeout=600000&connectTimeout=30000&allowMultiQueries=true&useLocalSessionState=true]]
[2023-11-30 18:50:15.800] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [[ALTER SESSION SET NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS', ALTER SESSION SET NLS_TIMESTAMP_FORMAT='YYYY-MM-DD HH24:MI:SS.FF', ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT='YYYY-MM-DD HH24:MI:SS.FF TZR TZD', ALTER SESSION SET TIME_ZONE='+00:00'] for instance 10.186.16.126:1521/EE.ORACLE.DOCKER user OBTEST characterSet utf8]
[2023-11-30 18:50:15.806] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [Oracle properties: {oracle.jdbc.ReadTimeout=-1}]
[2023-11-30 18:50:15.806] [com.oceanbase.oms.dataflow.datasource.DataSourceFactory] [INFO] [main] [create 10.186.16.126:1521/EE.ORACLE.DOCKER|OBTEST|UTF8 data source success , charset : utf8 , sql timeout : -1 , {sourceJdbcTableCache=false, forceRowid=false, sliceByMarcroinfo=false, type=DATAFLOW_SOURCE, binlogOffsetTable=binlog_offset, maxConnNum=32, jar=connector-dataflow.jar, ncharCharsetMap={"AL16UTF16":"UTF16"}, sliceUseCondition=false, indexHint=true, sliceBatchSize=600, nullReplaceEnable=false, oracleRowidSliceHint=, sourceMaxRetryTimes=3, sourceQueueSize=2000, rtrimDB2Graphic=false, sql_select_limit=9223372036854775807, sliceQueueSize=128, consistencyMigration=false, oracleRowidSliceRowsMin=-1, maxSqlRetryTime=3600, boosterClass=com.oceanbase.oms.connector.source.dataflow.DataFlowSourceBooster, charset=utf8, sliceBlocks=128, enableOmsConnectorV2Report=true, sliceWorkerNum=8, clients=[{"password":"111111","clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","instance":"10.186.16.126:1521/EE.ORACLE.DOCKER","username":"OBTEST"}], stringOriginalCharset=, timezone=+08:00, databaseMaxConnection=50, databaseConnectTimeout=120, sourceBatchMemorySize=16777216, binlogOffsetDb=test, intervalTypeFmt=, sliceOracleOptimizerFeatures=, taskResume=false, connTimeoutSec=120, maxSqlNoActiveTime=700, fetchSize=1000, nonePkUkQueryWholeTable=true, dbType=ORACLE, nullReplaceString= , rtrimOracleChar=false, timestamptzConvert=false, retryIntervalTime=3, nonePkUkTruncateDstTable=true, filterTmpTable=true, enableSplitByPartition=true, workerNum=8, batchSize=512, sliceByPkIncrementMinMax=true, maxActive=50}]
[2023-11-30 18:50:16.028] [com.alibaba.druid.pool.DruidDataSource] [INFO] [main] [{dataSource-1} inited]
...
 
获取表的元信息 - 判断白名单规则: 
[2023-11-30 18:50:16.684] [com.oceanbase.oms.connector.source.dataflow.ConditionTableSearcher] [INFO] [main] [ignore tableWhiteLists config null, use conditions config]
...
 
获取源库的元信息 - 获取配置:
[2023-11-30 18:50:16.685] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient] [INFO] [main] [SELECT VALUE FROM NLS_DATABASE_PARAMETERS WHERE PARAMETER='NLS_CHARACTERSET']
[2023-11-30 18:50:16.718] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient] [INFO] [main] [SELECT VALUE FROM NLS_DATABASE_PARAMETERS WHERE PARAMETER='NLS_NCHAR_CHARACTERSET']
 
获取源库的元信息 - 获取表名:
[2023-11-30 18:50:16.721] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient] [INFO] [main] [SELECT NULL,A.OWNER,A.TABLE_NAME,A.NUM_ROWS,A.PARTITIONED,A.IOT_TYPE FROM ALL_TABLES A WHERE (A.OWNER,A.TABLE_NAME) NOT IN ( SELECT OWNER,MVIEW_NAME FROM ALL_MVIEWS UNION ALL SELECT LOG_OWNER,LOG_TABLE FROM ALL_MVIEW_LOGS )  AND A.OWNER NOT IN ('SYSTEM','SYS')  AND A.TEMPORARY='N' ]
...
 
获取源库的元信息 - 获取列名: 
[2023-11-30 18:50:19.031] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient] [INFO] [main] [SELECT OWNER,TABLE_NAME,COLUMN_NAME FROM ALL_TAB_COLUMNS WHERE ((OWNER='OBTEST' AND TABLE_NAME='ORDER_LINE') OR (OWNER='OBTEST' AND TABLE_NAME='STOCK') OR (OWNER='OBTEST' AND TABLE_NAME='WAREHOUSE') OR (OWNER='OBTEST' AND TABLE_NAME='ITEM') OR (OWNER='OBTEST' AND TABLE_NAME='DISTRICT') OR (OWNER='OBTEST' AND TABLE_NAME='HISTORY') OR (OWNER='OBTEST' AND TABLE_NAME='ORDERS') OR (OWNER='OBTEST' AND TABLE_NAME='CUSTOMER') OR (OWNER='OBTEST' AND TABLE_NAME='NEW_ORDER'))]
...
 
获取源库的元信息 - 结果:
[2023-11-30 18:50:21.390] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [10.186.16.126:1521/EE.ORACLE.DOCKER OBTEST.ORDER_LINE IndexName: [IORDL] , type : PRIMARY , columns : [OL_W_ID,OL_D_ID,OL_O_ID,OL_NUMBER]]
[2023-11-30 18:50:21.390] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [10.186.16.126:1521/EE.ORACLE.DOCKER OBTEST.STOCK IndexName: [STOCK_I1] , type : UNIQUE_NULLABLE , columns : [S_I_ID,S_W_ID]]
[2023-11-30 18:50:21.391] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [10.186.16.126:1521/EE.ORACLE.DOCKER OBTEST.WAREHOUSE IndexName: [WAREHOUSE_I1] , type : UNIQUE_NULLABLE , columns : [W_ID]]
[2023-11-30 18:50:21.392] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [10.186.16.126:1521/EE.ORACLE.DOCKER OBTEST.ITEM IndexName: [ITEM_I1] , type : UNIQUE_NULLABLE , columns : [I_ID]]
[2023-11-30 18:50:21.392] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [10.186.16.126:1521/EE.ORACLE.DOCKER OBTEST.DISTRICT IndexName: [DISTRICT_I1] , type : UNIQUE_NULLABLE , columns : [D_W_ID,D_ID]]
...
 
初始化目标库的连接: 
[2023-11-30 18:50:21.495] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [init druid dataSource with loginTimeout[300], maxWaitMillis[1200000], maxActive[128], initActive[0], minIdle[0]]
[2023-11-30 18:50:21.496] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [connect with url [jdbc:oceanbase://10.186.16.126:32883?useUnicode=true&rewriteBatchedStatements=true&connectTimeout=30000&socketTimeout=600000&characterEncoding=utf8&sendConnectionAttributes=false&useServerPrepStmts=false]]
[2023-11-30 18:50:21.496] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [[ALTER SESSION SET NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS', ALTER SESSION SET NLS_TIMESTAMP_FORMAT='YYYY-MM-DD HH24:MI:SS.FF', ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT='YYYY-MM-DD HH24:MI:SS.FF TZR TZD', set ob_org_cluster_id = 2147473648, SET TIME_ZONE='+00:00';, ALTER SESSION SET sql_select_limit = 9223372036854775807;] for instance 10.186.16.126:32883 user OBTEST@oms_oracle#obthree characterSet utf8]
[2023-11-30 18:50:21.499] [com.oceanbase.oms.dataflow.datasource.DataSourceFactory] [INFO] [main] [create 10.186.16.126:32883|OBTEST@oms_oracle#obthree|UTF8 data source success , charset : utf8 , sql timeout : -1 , {sql_select_limit=9223372036854775807, testWhileIdle=true, testOnBorrow=true, timeBetweenEvictionRunsMillis=2000, minEvictableIdleTimeMillis=600000, localRegionNo=2147473648, testOnReturn=true, maxWait=1200000, loginTimeout=300, maxActive=128}]
[2023-11-30 18:50:21.502] [com.alibaba.druid.pool.DruidDataSource] [INFO] [main] [{dataSource-2} inited]
...
 
获取目标库的配置/库表信息等 (参考源库):
[2023-11-30 18:50:21.774] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oboracle.OBOracleJDBCClient] [INFO] [main] [SELECT VALUE FROM NLS_DATABASE_PARAMETERS WHERE PARAMETER='NLS_CHARACTERSET']
[2023-11-30 18:50:21.791] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oboracle.OBOracleJDBCClient] [INFO] [main] [NLS_CHARACTERSET=AL32UTF8 use charset utf8 for ob oracle mode]
[2023-11-30 18:50:21.792] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient] [INFO] [main] [SELECT VALUE FROM NLS_DATABASE_PARAMETERS WHERE PARAMETER='NLS_NCHAR_CHARACTERSET']
[2023-11-30 18:50:21.801] [com.oceanbase.oms.dataflow.jdbcclient.AbstractJDBCClient] [INFO] [main] [SHOW VARIABLES LIKE 'version_comment']
......
 
 
确定切片方式: 
[2023-11-30 18:50:22.109] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.ORDER_LINE slice by oracle block]
[2023-11-30 18:50:22.109] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.STOCK slice by oracle block]
[2023-11-30 18:50:22.110] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.ORDERS slice by oracle block]
[2023-11-30 18:50:22.110] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.DISTRICT slice by oracle block]
[2023-11-30 18:50:22.110] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.WAREHOUSE slice by oracle block]
[2023-11-30 18:50:22.110] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.CUSTOMER slice by oracle block]
[2023-11-30 18:50:22.111] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.ITEM slice by oracle block]
[2023-11-30 18:50:22.111] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.HISTORY slice by oracle block]
[2023-11-30 18:50:22.111] [com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor] [INFO] [main] [OBTEST.NEW_ORDER slice by oracle block]
 
源-目标 的表映射关系: 
[2023-11-30 18:50:22.158] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	whiteCondition = [{"all":false,"sub":[{"name":"CUSTOMER","map":"CUSTOMER"},{"name":"DISTRICT","map":"DISTRICT"},{"name":"HISTORY","map":"HISTORY"},{"name":"ITEM","map":"ITEM"},{"name":"NEW_ORDER","map":"NEW_ORDER"},{"name":"ORDERS","map":"ORDERS"},{"name":"ORDER_LINE","map":"ORDER_LINE"},{"name":"STOCK","map":"STOCK"},{"name":"WAREHOUSE","map":"WAREHOUSE"}],"name":"OBTEST","map":"OBTEST"}]]
 
源端的配置: 
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [[source]]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	boosterClass = com.oceanbase.oms.connector.source.dataflow.DataFlowSourceBooster]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	enableOmsConnectorV2Report = true]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	clients = [{"password":"111111","clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER","instance":"10.186.16.126:1521/EE.ORACLE.DOCKER","username":"OBTEST"}]]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sliceBatchSize = 600]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sliceWorkerNum = 8]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sourceJdbcTableCache = true]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	timezone = +08:00]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	dbType = ORACLE]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	databaseMaxConnection = 50]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	type = DATAFLOW_SOURCE]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sourceBatchMemorySize = 16777216]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	filterTmpTable = true]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	jar = connector-dataflow.jar]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	workerNum = 8]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	taskResume = false]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	batchSize = 512]
 
定制构建器的配置: 
[2023-11-30 18:50:22.158] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [[coordinator]]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	caseStrategy = follow-source]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	taskIdentity = np_59zi2w5dahgg]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	enableOmsConnectorV2Report = true]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	timezone = +08:00]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	enableActiveReportTask = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	taskSubId = 1]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	checkDstTableEmpty = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	bridgeQueueSize = 128]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	enableMetricReportTask = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	customizeClass = com.oceanbase.oms.connector.fullcustomize.FullFrameworkCustomize]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sourceType = ORACLE]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	throttleMemoryBound = 2147483648]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	printTraceMsg = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sinkType = OB_ORACLE]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	taskResume = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	listenPort = 16001]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	ignoreCompensateDDL = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	connectorJvmParam = -server -Xms8g -Xmx8g -Xmn4g -Xss512k]
 
目标端的配置: 
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [[sink]]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	boosterClass = com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSinkBooster]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	caseStrategy = follow-source]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	enableNoUniqueConstraintTableReplicate = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	accurateInvalidateCache = true]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	localRegionNo = 2147473648]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	timezone = +08:00]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	filterHiddenPK = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	isPreLoadAllTableSchema = false]
[2023-11-30 18:50:22.159] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	writeMode = full]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	type = JDBC_SINK_OB_ORACLE]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	useSourceRecord = true]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	password = 111111]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	canLobBatched = false]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	jdbcUrl = jdbc:oceanbase://10.186.16.126:32883?useUnicode=true&allowMultiQueries=true&socketTimeout=50000&characterEncoding=utf8&sendConnectionAttributes=false]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sinkMaps = [["OBTEST","HISTORY"],["OBTEST","STOCK"],["OBTEST","CUSTOMER"],["OBTEST","NEW_ORDER"],["OBTEST","ORDER_LINE"],["OBTEST","WAREHOUSE"],["OBTEST","ORDERS"],["OBTEST","DISTRICT"],["OBTEST","ITEM"]]]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	jar = jdbc-sink-ob-oracle.jar]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	workerNum = 8]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	sinkType = OB_ORACLE]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	forceUpdateTimeField = false]
[2023-11-30 18:50:22.160] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [	username = OBTEST@oms_oracle#obthree]
 
关闭上述使用的源端/目标端的连接: 
[2023-11-30 18:50:22.186] [com.alibaba.druid.pool.DruidDataSource] [INFO] [main] [{dataSource-2} closing ...]
[2023-11-30 18:50:22.189] [com.alibaba.druid.pool.DruidDataSource] [INFO] [main] [{dataSource-2} closed]
[2023-11-30 18:50:22.189] [com.alibaba.druid.pool.DruidDataSource] [INFO] [main] [{dataSource-1} closing ...]
[2023-11-30 18:50:22.196] [com.alibaba.druid.pool.DruidDataSource] [INFO] [main] [{dataSource-1} closed]
 
 
创建切片任务: 
[2023-11-30 18:50:22.198] [com.oceanbase.oms.connector.source.dataflow.SliceTaskManager] [INFO] [main] [slice table in order 0.OBTEST.CUSTOMER]
[2023-11-30 18:50:22.198] [com.oceanbase.oms.connector.source.dataflow.SliceTaskManager] [INFO] [main] [slice table in order 1.OBTEST.DISTRICT]
[2023-11-30 18:50:22.198] [com.oceanbase.oms.connector.source.dataflow.SliceTaskManager] [INFO] [main] [slice table in order 2.OBTEST.HISTORY]
...
[2023-11-30 18:50:22.297] [com.oceanbase.oms.dataflow.slice.SliceFactory] [INFO] [main] [create SliceProvider success : local]
[2023-11-30 18:50:22.298] [com.oceanbase.oms.connector.source.dataflow.DataFlowSliceTask] [INFO] [main] [OBTEST.CUSTOMER start checkpoint. null]
[2023-11-30 18:50:22.301] [com.oceanbase.oms.dataflow.slice.SliceFactory] [INFO] [main] [create SliceProvider success : local]
[2023-11-30 18:50:22.302] [com.oceanbase.oms.connector.source.dataflow.DataFlowSliceTask] [INFO] [main] [OBTEST.DISTRICT start checkpoint. null]
[2023-11-30 18:50:22.305] [com.oceanbase.oms.dataflow.slice.SliceFactory] [INFO] [main] [create SliceProvider success : local]
[2023-11-30 18:50:22.305] [com.oceanbase.oms.connector.source.dataflow.DataFlowSliceTask] [INFO] [main] [OBTEST.HISTORY start checkpoint. null]
...
 
创建新的目标库的连接: 
2023-11-30 18:50:22.326] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [init druid dataSource with loginTimeout[300], maxWaitMillis[1200000], maxActive[8], initActive[8], minIdle[8]]
[2023-11-30 18:50:22.326] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [connect with url [jdbc:oceanbase://10.186.16.126:32883?useUnicode=true&rewriteBatchedStatements=true&socketTimeout=50000&connectTimeout=30000&allowMultiQueries=true&characterEncoding=utf8&sendConnectionAttributes=false&useServerPrepStmts=false]]
[2023-11-30 18:50:22.326] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [[ALTER SESSION SET NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS', ALTER SESSION SET NLS_TIMESTAMP_FORMAT='YYYY-MM-DD HH24:MI:SS.FF', ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT='YYYY-MM-DD HH24:MI:SS.FF TZR TZD', set ob_org_cluster_id = 2147473648, SET TIME_ZONE='+08:00';, ALTER SESSION SET sql_select_limit = 9223372036854775807;] for instance 10.186.16.126:32883 user OBTEST@oms_oracle#obthree characterSet ]
[2023-11-30 18:50:22.327] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [init druid dataSource with loginTimeout[300], maxWaitMillis[1200000], maxActive[3], initActive[0], minIdle[0]]
[2023-11-30 18:50:22.327] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [connect with url [jdbc:oceanbase://10.186.16.126:32883?useUnicode=true&rewriteBatchedStatements=true&socketTimeout=172800000&connectTimeout=30000&allowMultiQueries=true&characterEncoding=utf8&sendConnectionAttributes=false&useServerPrepStmts=false]]
[2023-11-30 18:50:22.327] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [main] [[ALTER SESSION SET NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS', ALTER SESSION SET NLS_TIMESTAMP_FORMAT='YYYY-MM-DD HH24:MI:SS.FF', ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT='YYYY-MM-DD HH24:MI:SS.FF TZR TZD', set ob_org_cluster_id = 2147473648, SET TIME_ZONE='+08:00';, ALTER SESSION SET sql_select_limit = 9223372036854775807;] for instance 10.186.16.126:32883 user OBTEST@oms_oracle#obthree characterSet ]
[2023-11-30 18:50:22.433] [com.alibaba.druid.pool.DruidDataSource] [INFO] [main] [{dataSource-3} inited]
 
主线程对目标库进行水位线跟踪 (OBHighWatermarkMonitor)
[2023-11-30 18:50:22.449] [com.oceanbase.oms.dataflow.jdbcclient.AbstractJDBCClient] [INFO] [main] [SHOW VARIABLES LIKE 'version_comment']
[2023-11-30 18:50:22.518] [com.oceanbase.oms.connector.common.ob.OBHighWatermarkMonitor] [INFO] [main] [Inited OBHighWatermarkMonitor. querySql. [OB_ORACLE], cpuSQL: select /*+query_timeout(5000000)*/ 100 - max(round(memstore_used/memstore_limit*100)) from SYS.GV$OB_MEMSTORE, memSQL: select /*+query_timeout(5000000)*/ 100 - max(round(memstore_used/memstore_limit*100)) from SYS.GV$OB_MEMSTORE]
 
启动 切分器的线程:
[2023-11-30 18:50:22.523] [com.oceanbase.oms.connector.source.dataflow.SliceTaskManager] [INFO] [slice-worker-0] [slice-worker-0 start slice task]
[2023-11-30 18:50:22.523] [com.oceanbase.oms.connector.source.dataflow.SliceTaskManager] [INFO] [slice-worker-1] [slice-worker-1 start slice task]
[2023-11-30 18:50:22.523] [com.oceanbase.oms.connector.source.dataflow.SliceTaskManager] [INFO] [slice-worker-2] [slice-worker-2 start slice task]
[2023-11-30 18:50:22.523] [com.oceanbase.oms.connector.source.dataflow.SliceTaskManager] [INFO] [slice-worker-3] [slice-worker-3 start slice task]
...
 
初始化recordDispatcher:
[2023-11-30 18:50:22.582] [com.oceanbase.oms.connector.dispatcher.shard.AbstractShardDispatcher] [INFO] [main] [DynamicShardDispatcher: start with capacity [16384], minBatchSize [20]]
[2023-11-30 18:50:22.582] [com.oceanbase.oms.connector.dispatcher.shard.AbstractShardDispatcher] [INFO] [main] [DynamicShardDispatcher: start with batchMode, flushBatchInterval [5000]]

 
初始化各文件目录 (状态显示/checkpoint管理/限流机制):
[2023-11-30 18:50:22.634] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [BootStrap: init memoryBoound value to 2147483648]
[2023-11-30 18:50:22.670] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [BootStrapPanel: register keep alive url [/status]]
[2023-11-30 18:50:22.672] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [BootStrapPanel: register source stats url [/source/stats]]
[2023-11-30 18:50:22.673] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [BootStrapPanel: register checkpoint query url [/checkpoint/query]]
[2023-11-30 18:50:22.674] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [BootStrapPanel: register throttle control url [/throttle/control]]
[2023-11-30 18:50:22.675] [com.oceanbase.oms.connector.coordinator.BootStrapPanel] [INFO] [main] [BootStrapPanel: register throttle query url [/throttle/query]]
 
 
初始化多个Writer (sinkTask):
[2023-11-30 18:50:22.701] [com.oceanbase.oms.connector.jdbc.sink.Writer] [INFO] [main] [Writer init with arguments, write mode[FULL_BATCH_PREPARE_STATEMENT], maxTxnRetryTimes[1000], retryIntervalTime[1000],dryRun[false], printSql[false], slowSqlThreshold[15], ignoreDDL[true], hotKeyMerge[false]typeStatistics[false], fullBatchWithSameTable[false], accurateInvalidateCache[true], canLobBatched[false]]
[2023-11-30 18:50:22.701] [com.oceanbase.oms.connector.jdbc.sink.batch.SemiPrepareStatementBatchAccumulate] [INFO] [main] [FullPrepareStatementBatchAccumulate created, can batch lob [false]]
...
 
初始化多个任务线程:
[2023-11-30 18:50:22.713] [com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask] [INFO] [main] [ConditionedSourceConnectorTask: start with maxRetryTime [1000], retryInterval [2000]]
[2023-11-30 18:50:22.716] [com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask] [INFO] [main] [ConditionedSourceConnectorTask: start with maxRetryTime [1000], retryInterval [2000]]
...
 
切分器线程, 初始化链接: 
[2023-11-30 18:50:23.530] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [slice-worker-2] [init druid dataSource with loginTimeout[300], maxWaitMillis[1200000], maxActive[50], initActive[0], minIdle[0]]
[2023-11-30 18:50:23.530] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [slice-worker-2] [connect with url [jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER?rewriteBatchedStatements=true&GetConnectionTimeout=20000&socketTimeout=600000&connectTimeout=30000&allowMultiQueries=true&useLocalSessionState=true]]
[2023-11-30 18:50:23.531] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [slice-worker-2] [[ALTER SESSION SET NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS', ALTER SESSION SET NLS_TIMESTAMP_FORMAT='YYYY-MM-DD HH24:MI:SS.FF', ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT='YYYY-MM-DD HH24:MI:SS.FF TZR TZD', ALTER SESSION SET TIME_ZONE='+08:00'] for instance 10.186.16.126:1521/EE.ORACLE.DOCKER user OBTEST characterSet AL32UTF8]
[2023-11-30 18:50:23.531] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [slice-worker-2] [Oracle properties: {oracle.jdbc.ReadTimeout=-1}]
[2023-11-30 18:50:23.531] [com.oceanbase.oms.dataflow.datasource.DataSourceFactory] [INFO] [slice-worker-2] [create 10.186.16.126:1521/EE.ORACLE.DOCKER|OBTEST|AL32UTF8 data source success , charset : AL32UTF8 , sql timeout : -1 , {limitator.oom.avoid=false, sql_select_limit=9223372036854775807, datasource.timezone=+08:00, maxActive=50}]
[2023-11-30 18:50:23.536] [com.alibaba.druid.pool.DruidDataSource] [INFO] [slice-worker-2] [{dataSource-4} inited]
...
 
切分器线程, 获取BLOCK:
[2023-11-30 18:50:23.580] [com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient] [INFO] [slice-worker-2] [SELECT  OMS_OBJECT_NUMBER, OMS_RELATIVE_FNO, OMS_BLOCK_NUMBER, OMS_ROW_NUMBER, BLOCKS FROM ( SELECT OBJ.DATA_OBJECT_ID OMS_OBJECT_NUMBER,EXT.RELATIVE_FNO OMS_RELATIVE_FNO, EXT.BLOCK_ID OMS_BLOCK_NUMBER, 0 OMS_ROW_NUMBER , EXT.BLOCKS BLOCKS   FROM DBA_EXTENTS EXT, ALL_OBJECTS OBJ WHERE EXT.OWNER = 'OBTEST' AND EXT.OWNER = OBJ.OWNER AND EXT.SEGMENT_NAME = 'HISTORY' AND EXT.SEGMENT_TYPE IN ('TABLE', 'TABLE PARTITION','TABLE SUBPARTITION') AND EXT.SEGMENT_NAME = OBJ.OBJECT_NAME   ) ORDER BY OMS_OBJECT_NUMBER, OMS_RELATIVE_FNO, OMS_BLOCK_NUMBER]

切分器线程, 生成切分片
[2023-11-30 18:50:23.678] [com.oceanbase.oms.dataflow.slice.BlockSliceService] [INFO] [slice-worker-2] [{OBTEST.HISTORY BLOCK 1 [null] - [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:18152,OMS_ROW_NUMBER:0]}]
[2023-11-30 18:50:23.678] [com.oceanbase.oms.connector.source.dataflow.CheckpointStore] [INFO] [slice-worker-2] [addCheckpoint: {OBTEST.HISTORY BLOCK 1 [null] - [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:18152,OMS_ROW_NUMBER:0]}]

[2023-11-30 18:50:23.680] [com.oceanbase.oms.dataflow.slice.BlockSliceService] [INFO] [slice-worker-2] [{OBTEST.HISTORY BLOCK 2 [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:18152,OMS_ROW_NUMBER:0] - [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:50600,OMS_ROW_NUMBER:0]}]
[2023-11-30 18:50:23.681] [com.oceanbase.oms.connector.source.dataflow.CheckpointStore] [INFO] [slice-worker-2] [addCheckpoint: {OBTEST.HISTORY BLOCK 2 [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:18152,OMS_ROW_NUMBER:0] - [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:50600,OMS_ROW_NUMBER:0]}]
 
...
 
切分器线程, 一次切分任务完成: 
[2023-11-30 18:50:23.697] [com.oceanbase.oms.dataflow.slice.BlockSliceService] [INFO] [slice-worker-2] [slice finish ,total : 33 , prepstmt:0.444,getrow:{cnt:32,totms:25.441,avgms:0.795,minms:0.215,maxms:9.357},setparam:0.002,exequery:88.210,getconn:42.327]
...
 
源库线程, 获取源库的数据:
[2023-11-30 18:50:23.819] [com.oceanbase.oms.dataflow.datasource.druid.AbstractDruidDataSource] [INFO] [sourceTask-0] [ALTER SESSION SET TIME_ZONE='+08:00']
[2023-11-30 18:50:23.819] [com.oceanbase.oms.dataflow.jdbcclient.AbstractQueryClient] [INFO] [sourceTask-0] [{OBTEST.HISTORY BLOCK 6 [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:50968,OMS_ROW_NUMBER:0] - [OMS_OBJECT_NUMBER:76318,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:50992,OMS_ROW_NUMBER:0]},SELECT /*+ ROWID("HISTORY") */ ROWID,"H_C_ID","H_C_D_ID","H_C_W_ID","H_D_ID","H_W_ID","H_DATE","H_AMOUNT","H_DATA" FROM "OBTEST"."HISTORY"  WHERE ROWID>? AND ROWID<=?] 
...
 
 
Sink线程, 回放数据:
[2023-11-30 18:50:24.294] [com.oceanbase.oms.connector.source.dataflow.CheckpointStore] [INFO] [sinkTask-7] [ackCheckpoint: {OBTEST.ITEM BLOCK 2 [OMS_OBJECT_NUMBER:76320,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:11464,OMS_ROW_NUMBER:0] - [OMS_OBJECT_NUMBER:76320,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:11480,OMS_ROW_NUMBER:0]}]
[2023-11-30 18:50:24.294] [com.oceanbase.oms.connector.source.dataflow.AckBatchListener] [INFO] [sinkTask-7] [OBTEST.ITEM {OBTEST.ITEM BLOCK 2 [OMS_OBJECT_NUMBER:76320,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:11464,OMS_ROW_NUMBER:0] - [OMS_OBJECT_NUMBER:76320,OMS_RELATIVE_FNO:5,OMS_BLOCK_NUMBER:11480,OMS_ROW_NUMBER:0]} rows : 474 , onNewBatchState:0.316]

心跳信息: 
[2023-11-30 18:54:32.691] [com.oceanbase.oms.connector.source.dataflow.OverviewManager] [INFO] [timerTaskScheduler] [overview refresh start]
[2023-11-30 18:54:32.691] [com.oceanbase.oms.connector.source.dataflow.OverviewManager] [INFO] [timerTaskScheduler] [overview refresh end]
[2023-11-30 18:54:32.691] [com.oceanbase.oms.connector.source.dataflow.HeartBeatManager] [INFO] [timerTaskScheduler] [migrate/1/heartbeat : {"accumulatedRunningTime":257,"capacit
y":5089999,"consistentQuantity":0,"dstRps":20216,"dstRpsRef":8000,"dstRt":0,"dstRtRef":1,"dstSpeed":16535159,"failedTables":0,"finishedTables":8,"firstRoundStartingGmtTime":17013
41415,"id":"1","inconsistentQuantity":0,"message":"","numberOfDeletedRecords":0,"numberOfInsertedRecords":5022315,"numberOfUpdatedRecords":0,"predictedTimeToFinish":3,"processedR
ecords":5022315,"progress":"0.889","recordProgress":"0.987","reportingGmtTime":1701341672,"rps":20216,"srcRps":20216,"srcRpsRef":8000,"srcRt":0,"srcRtRef":1,"srcSpeed":16535159,"
srcSpeedRef":8388608,"startingGmtTime":1701341415,"status":"running","subId":"1","totalTables":9,"type":"migrate"}]
```
