---
title: 20231127 - 在Oracle并发压力中, 测试OMS性能 [4]
confluence_page_id: 2589728
created_at: 2023-11-27T03:59:22+00:00
updated_at: 2023-11-28T08:10:51+00:00
---

# 背景

  - 场景: 10 warehouse, 10 vu, 仅做buildschema数据初始化, 数据同步慢, 进行如下调优: 

```
diset tpcc count_ware 10
diset tpcc num_vu 10
```

  - 调整源端 logfetcher SQL的间隔: 

```
deliver2store.logminer.fetch_online_log_interval_seconds = 1
```

  - 调整目标端配置, 放大队列13的准入阈值:

```
coordinator.maxRecordCapacity = 1000000
```

  - 调整目标端java程序的运行内存:

```
coordinator.connectorJvmParam = -Xms10g -Xmx10g ...
```

  - 调整目标端java的printSinkRecords的行为: 

```
options strict false
vmtool --action getInstances --className com.oceanbase.oms.connector.common.log.MsgUtils --express 'instances[0].printTraceMsg = false'
options strict true
```

  - 调整目标端 sink.workerNum = 128

  - 调整源端 deliver2store.logminer.converter_thread_num = 16

  - 使用ConflictBrokerV2

    - 在目标端java元数据中, 增加: 

`coordinator.hotKeyMerge = ``true``sink.hotKeyTable = [{"schema":``"OBTEST"``, "tableName":``"WAREHOUSE"``},{"schema":``"OBTEST"``, "tableName":``"DISTRICT"``}]`  
---  
  
  - 为了避免IoT表, 手工创建TPCC表: 

```
  CREATE TABLE "OBTEST"."ITEM"
   (	"I_ID" NUMBER(6,0),
	"I_IM_ID" NUMBER,
	"I_NAME" VARCHAR2(24),
	"I_PRICE" NUMBER(5,2),
	"I_DATA" VARCHAR2(50)
   ) INITRANS 4 PCTFREE 10;
 
CREATE TABLE "OBTEST"."NEW_ORDER"
   (	"NO_W_ID" NUMBER,
	"NO_D_ID" NUMBER,
	"NO_O_ID" NUMBER,
	 CONSTRAINT "INORD" PRIMARY KEY ("NO_W_ID", "NO_D_ID", "NO_O_ID") ENABLE
   ) NOCOMPRESS INITRANS 4 PCTFREE 10;
 
CREATE TABLE "OBTEST"."ORDER_LINE"
   (	"OL_W_ID" NUMBER,
	"OL_D_ID" NUMBER,
	"OL_O_ID" NUMBER,
	"OL_NUMBER" NUMBER,
	"OL_I_ID" NUMBER,
	"OL_DELIVERY_D" DATE,
	"OL_AMOUNT" NUMBER,
	"OL_SUPPLY_W_ID" NUMBER,
	"OL_QUANTITY" NUMBER,
	"OL_DIST_INFO" CHAR(24),
	 CONSTRAINT "IORDL" PRIMARY KEY ("OL_W_ID", "OL_D_ID", "OL_O_ID", "OL_NUMBER") ENABLE
   ) NOCOMPRESS INITRANS 4 PCTFREE 10;
 
CREATE TABLE "OBTEST"."ORDERS"
   (	"O_ID" NUMBER,
	"O_W_ID" NUMBER,
	"O_D_ID" NUMBER,
	"O_C_ID" NUMBER,
	"O_CARRIER_ID" NUMBER,
	"O_OL_CNT" NUMBER,
	"O_ALL_LOCAL" NUMBER,
	"O_ENTRY_D" DATE
   ) INITRANS 4 PCTFREE 10;
 
 CREATE TABLE "OBTEST"."STOCK"
   (	"S_I_ID" NUMBER(6,0),
	"S_W_ID" NUMBER(6,0),
	"S_QUANTITY" NUMBER(6,0),
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
	"S_YTD" NUMBER(10,0),
	"S_ORDER_CNT" NUMBER(6,0),
	"S_REMOTE_CNT" NUMBER(6,0),
	"S_DATA" VARCHAR2(50)
   ) INITRANS 4 PCTFREE 10;
 
CREATE TABLE "OBTEST"."WAREHOUSE"
   (	"W_ID" NUMBER(6,0),
	"W_YTD" NUMBER(12,2),
	"W_TAX" NUMBER(4,4),
	"W_NAME" VARCHAR2(10),
	"W_STREET_1" VARCHAR2(20),
	"W_STREET_2" VARCHAR2(20),
	"W_CITY" VARCHAR2(20),
	"W_STATE" CHAR(2),
	"W_ZIP" CHAR(9)
   ) INITRANS 4 PCTFREE 99 PCTUSED 1;
 
CREATE TABLE "OBTEST"."HISTORY"
   (	"H_C_ID" NUMBER,
	"H_C_D_ID" NUMBER,
	"H_C_W_ID" NUMBER,
	"H_D_ID" NUMBER,
	"H_W_ID" NUMBER,
	"H_DATE" DATE,
	"H_AMOUNT" NUMBER(6,2),
	"H_DATA" VARCHAR2(24)
   ) INITRANS 4 PCTFREE 10;
 
  CREATE TABLE "OBTEST"."DISTRICT"
   (	"D_ID" NUMBER(2,0),
	"D_W_ID" NUMBER(6,0),
	"D_YTD" NUMBER(12,2),
	"D_TAX" NUMBER(4,4),
	"D_NEXT_O_ID" NUMBER,
	"D_NAME" VARCHAR2(10),
	"D_STREET_1" VARCHAR2(20),
	"D_STREET_2" VARCHAR2(20),
	"D_CITY" VARCHAR2(20),
	"D_STATE" CHAR(2),
	"D_ZIP" CHAR(9)
   )INITRANS 4 PCTFREE 99 PCTUSED 1;
 
 CREATE TABLE "OBTEST"."CUSTOMER"
   (	"C_ID" NUMBER(5,0),
	"C_D_ID" NUMBER(2,0),
	"C_W_ID" NUMBER(6,0),
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
	"C_PAYMENT_CNT" NUMBER(8,0),
	"C_DELIVERY_CNT" NUMBER(8,0),
	"C_DATA" VARCHAR2(500)
   ) INITRANS 4 PCTFREE 10;
 
 
CREATE UNIQUE INDEX "OBTEST"."WAREHOUSE_I1" ON "OBTEST"."WAREHOUSE" ("W_ID");
CREATE UNIQUE INDEX "OBTEST"."STOCK_I1" ON "OBTEST"."STOCK" ("S_I_ID", "S_W_ID");
CREATE UNIQUE INDEX "OBTEST"."ORDERS_I1" ON "OBTEST"."ORDERS" ("O_W_ID", "O_D_ID", "O_ID");
CREATE UNIQUE INDEX "OBTEST"."ORDERS_I2" ON "OBTEST"."ORDERS" ("O_W_ID", "O_D_ID", "O_C_ID", "O_ID");
CREATE UNIQUE INDEX "OBTEST"."ITEM_I1" ON "OBTEST"."ITEM" ("I_ID");
CREATE UNIQUE INDEX "OBTEST"."DISTRICT_I1" ON "OBTEST"."DISTRICT" ("D_W_ID", "D_ID");
CREATE UNIQUE INDEX "OBTEST"."CUSTOMER_I1" ON "OBTEST"."CUSTOMER" ("C_W_ID", "C_D_ID", "C_ID");
CREATE UNIQUE INDEX "OBTEST"."CUSTOMER_I2" ON "OBTEST"."CUSTOMER" ("C_LAST", "C_W_ID", "C_D_ID", "C_FIRST", "C_ID");
 
ALTER TABLE OBTEST.NEW_ORDER ADD SUPPLEMENTAL LOG GROUP LGN1700755628055 (NO_D_ID,NO_O_ID,NO_W_ID) ALWAYS; 
ALTER TABLE OBTEST.ORDER_LINE ADD SUPPLEMENTAL LOG GROUP LGN1700755628056 (OL_NUMBER,OL_W_ID,OL_O_ID,OL_D_ID) ALWAYS; 
```

# 测试

队列情况

```
huangyan@R740-26:~$ ./arthas_get_queue_status.sh
../../arthas-boot.jar
[
  "1: LogEntryFetcher_OutQueue_Size=6",
  "2: OracleLogExtractor_OutQueue_Size=0",
  "3: LogRecordSerializer_OutQueue_Size=0",
  "4: LogRecordConverterPrepare_OutQueue_Size=1216",
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
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=975720/1000000",
  "13: TransactionScheduler_OutQueue_Size=7508",
  "sinkTasksMax=128"
]
"sinkTasksRunning="
3
[
  "ObConnectionRunning=22"
]
huangyan@R740-26:~$ ./arthas_get_queue_status.sh
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
  "11: StoreSource_OutQueue_Size=256",
  "12: ETLProcessor_OutQueue_Size=256",
  "13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=998853/1000000",
  "13: TransactionScheduler_OutQueue_Size=7691",
  "sinkTasksMax=128"
]
"sinkTasksRunning="
4
[
  "ObConnectionRunning=12"
]
``` 

问题集中于目标端回放并发不足, 判断有新的热点表产生? 

需要一个方法, 判断热点冲突的表:

```
watch com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV2 canMerge '{params[0], params[1], params[3].get(params[2]).size()}' 'params[3].get(params[2]).size() > 2'
``` 

未观察到热点冲突

在不同阶段, 观察到回放端饱和和回放端饿死 等几种情况

先诊断: 在源端事务 (50000数据/事务), 单个事务完成回放后, 好像目标端回放会停滞一段时间, 再开始上涨.

查看处理器12接到的事务大小, 大量是128数据/事务. 通过以下arthas命令检查, 128大小以外的事务较少

```
watch com.oceanbase.oms.connector.customize.batchprocessor.ETLBatchProcessor process 'params[0].size()' 'params[0].size()>0 && params[0].size() != 128'
``` 

事务切分器: 

```
com.oceanbase.oms.connector.batch.CommonTransactionAssembler.TransactionAssemblerInner#transactionCutter
 
构建位置: com.oceanbase.oms.connector.source.store.StoreSource#buildRecordListener
 
配置: 
source.splitThreshold, 事务切分的阈值, 默认值128个数据
source.sourceBatchMemorySize, 事务切分的内存阈值, 默认值为-1
``` 

更新目标端配置: source.splitThreshold = 4096, 再次测试

```
目标端队列, 呈现了并发不足的状况: 
@ArrayList[
    @String[11: StoreSource_OutQueue_Size=0],
    @String[12: ETLProcessor_OutQueue_Size=0],
    @String[13: TransactionScheduler_In_Rows/TransactionScheduler_In_MaxRows=498912/1000000],
    @String[13: TransactionScheduler_OutQueue_Size=12],
    @String[sinkTasksMax=128],
]
 
活跃连接数也很低: 
[arthas@72273]$ vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express '
> { "ObConnectionRunning=" + instances.{? #this.lock.isLocked()}.size()}
> '
@ArrayList[
    @String[ObConnectionRunning=10],
]
 
 
但没检测到冲突
watch com.oceanbase.oms.connector.dispatcher.scheduler.ConflictBrokerV2 canMerge '{params[0], params[1], params[3].get(params[2]).size()}' 'params[3].get(params[2]).size() >= 2'
无输出
``` 
    
    
     
    
    
    在目标数据库中看, INSERT INTO "OBTEST"."STOCK" 的并发是25个, 4k数据/事务, 总共是10w个数据, 符合原事务大小
    问题是, 原事务为什么不能和另一个10w数据的事务进行并发

观察事务切割器的提交行为: 

```
watch com.oceanbase.oms.connector.common.transaction.TransactionCutter finishOneTransaction
``` 

观察到的行为与猜测一致, 原事务不能并发, 导致供给不足

观察事务切割器的切割行为: 

```
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=21.260984ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=15.814354ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=16.628978ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=16.351682ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=16.344896ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=18.986721ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=15.634758ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=17.218605ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:18; [cost=16.035157ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:19; [cost=248.456336ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:19; [cost=16.085069ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:19; [cost=17.180887ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:19; [cost=3.525742ms] result=@Integer[849]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:19; [cost=0.036487ms] result=@Integer[1]
 
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=16.837133ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=15.061525ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=16.136495ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=17.371854ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=13.996234ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=15.402774ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=18.524019ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=14.98706ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=14.909515ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=14.877985ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=15.10539ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=15.885088ms] result=@Integer[4096]
method=com.oceanbase.oms.connector.batch.CommonTransactionAssembler$TransactionAssemblerInner.putTransaction location=AtExit
ts=2023-11-28 10:19:42; [cost=7.967525ms] result=@Integer[848]
``` 

对于一个事务的切割可以在几秒内完成, 但两个事务间的间隔过长

往上一环节诊断 (从store拉取数据): 

```
watch com.oceanbase.oms.connector.source.store.TransactionAssembler notify 'params[0].getRecordList().get(0).tableName' 'params[0].getRecordCount() > 0 && params[0].getRecordList().get(0).op == 7' 
 
从获取commit的时间戳: 
 
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:37:19; [cost=3.154301ms] result=null
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:37:44; [cost=3.280895ms] result=null
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:37:52; [cost=3.210318ms] result=null
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:37:59; [cost=3.030235ms] result=null
``` 

发现获取一个事务的完整间隔比较长, 诊断数据传输的效率: 

```
watch com.oceanbase.oms.connector.source.store.TransactionAssembler notify '{params[0].getRecordList().get(0).op, params[0].getRecordList().get(0).tableName, params[0].getRecordCount()}' 'params[0].getRecordCount() > 0' 
 
---
 
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:40:04; [cost=0.347873ms] result=@ArrayList[
    @Integer[0],
    @String[CUSTOMER],
    @Integer[1],
]
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:40:04; [cost=0.036922ms] result=@ArrayList[
    @Integer[0],
    @String[CUSTOMER],
    @Integer[1],
]
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:40:04; [cost=0.029531ms] result=@ArrayList[
    @Integer[0],
    @String[CUSTOMER],
    @Integer[1],
]
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:40:04; [cost=0.030466ms] result=@ArrayList[
    @Integer[0],
    @String[CUSTOMER],
    @Integer[1],
]
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:40:04; [cost=0.029581ms] result=@ArrayList[
    @Integer[0],
    @String[CUSTOMER],
    @Integer[1],
]
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:40:04; [cost=0.032042ms] result=@ArrayList[
    @Integer[0],
    @String[CUSTOMER],
    @Integer[1],
]
method=com.oceanbase.oms.connector.source.store.TransactionAssembler.notify location=AtExit
ts=2023-11-28 10:40:04; [cost=0.027399ms] result=@ArrayList[
    @Integer[0],
    @String[CUSTOMER],
    @Integer[1],
]
``` 

数据一直在传输, 但每次只传输一行数据, 每个数据的处理时间是0.03ms以上, 每秒只能传输3000行数据

检查DRC配置: 

```
[arthas@72273]$ vmtool --action getInstances --className com.oceanbase.oms.store.client.impl.DRCConfig --express 'instances[0]'
@DRCConfig[
    configures=@HashMap[isEmpty=false;size=14],
    userDefinedParams=@HashMap[isEmpty=false;size=6],
    persists=@HashSet[isEmpty=true;size=0],
    checkpoint=@Checkpoint[::::1701139878:],
    filter=@DataFilterV2[OBTEST.DISTRICT|OBTEST.ITEM|OBTEST.WAREHOUSE|OBTEST.ORDERS|OBTEST.CUSTOMER|OBTEST.ORDER_LINE|OBTEST.NEW_ORDER|OBTEST.STOCK|OBTEST.HISTORY|],
    blackList=@String[OBTEST.drc_txn*|OBTEST.DRC_TXN*],
    recordsPerBatch=@Integer[0],
...
    maxRecordsCached=@Integer[10240],
    maxRecordsBatched=@Integer[1024],
    maxTxnsBatched=@Integer[10240],
    maxTimeoutBatched=@Long[500],
    useDrcNet=@Boolean[true],
``` 

但检查store的配置时: 

配置代码为: 

```
com.oceanbase.oms.store.client.impl.ClusterManagers#connectStore
{
	...
	int num = config.getNumOfRecordsPerBatch();
	if (num > 0) {
	    server.addParam("writer.threshold", Integer.toString(num));
	}
	...
}
``` 

尝试强制修改配置: 

```
options strict false
vmtool --action getInstances --className com.oceanbase.oms.store.client.impl.DRCConfig --express 'instances[0].recordsPerBatch = 64'
options strict true
``` 

重启store

目标端重连到store的堆栈: 

```
    @com.oceanbase.oms.store.client.impl.ClusterManagers.connectStore()
        at com.oceanbase.oms.store.client.impl.ClusterManagers.connectStore(ClusterManagers.java:139)
        at com.oceanbase.oms.store.client.impl.ClusterManagers.findStore(ClusterManagers.java:105)
        at com.oceanbase.oms.store.client.impl.DRCClientImpl.connectClusterManager(DRCClientImpl.java:380)
        at com.oceanbase.oms.store.client.impl.DRCClientImpl.processRedirectMessage(DRCClientImpl.java:660)
        at com.oceanbase.oms.store.client.impl.DRCClientImpl.run(DRCClientImpl.java:446)
        at java.lang.Thread.run(Thread.java:853)
``` 

检查DRC client的配置: 

```
vmtool --action getInstances --className com.oceanbase.oms.store.client.impl.ServerProxy --express 'instances[0].handler.formParams'
 
[arthas@29848]$ vmtool --action getInstances --className com.oceanbase.oms.store.client.impl.ServerProxy --express 'instances[0].handler.formParams'
[{"name":"token","value":"dHM9MTcwMTE0OTQ3NjEyNyZjb25zdW1lcj1STV9vbXMmY29ubj0xMC4xODYuMTYuMTI2JTNBNTI0MzgtMTAuMTg2LjE2LjEyNiUzQTgwODg"},{"name":"checkpoint","value":"1701148927"},{"name":"writer.threshold","value":"64"},{"name":"useDrcNet","value":"true"},{"name":"writer.type","value":"sizedData"},{"name":"filter.blacklist","value":"OBTEST.drc_txn*|OBTEST.DRC_TXN*"},{"name":"filter.conditions","value":"OBTEST.DISTRICT|OBTEST.ITEM|OBTEST.WAREHOUSE|OBTEST.ORDERS|OBTEST.CUSTOMER|OBTEST.ORDER_LINE|OBTEST.NEW_ORDER|OBTEST.STOCK|OBTEST.HISTORY|"},{"name":"username","value":"RM_oms"},{"name":"password","value":"RM_oms"},{"name":"group","value":"RM_oms"},{"name":"subGroup","value":"ORACLE_np_59bb8o754l1c_59bbdrkasnc0-1-0"},{"name":"client.version","value":"58_SP"},{"name":"topic","value":"ORACLE_np_59bb8o754l1c_59bbdrkasnc0-1-0"}]
``` 

在store的日志/u01/ds/store/store7100/log/congo_0中, 查看参数已生效: 

```
2023-11-28 13:31:16 [WARN] [StoreManager.cpp:981 64524,570406656] get userdata: checkpoint=1701148927&filter.blacklist=OBTEST.drc_txn*|OBTEST.DRC_TXN*&subGroup=ORACLE_np_59bb8o754l1c_59bbdrkasnc0-1-0&client.version=58_SP&writer.threshold=64&writer.type=sizedData&useDrcNet=true&token=dHM9MTcwMTE0OTQ3NjEyNyZjb25zdW1lcj1STV9vbXMmY29ubj0xMC4xODYuMTYuMTI2JTNBNTI0MzgtMTAuMTg2LjE2LjEyNiUzQTgwODg&filter.conditions=OBTEST.DISTRICT|OBTEST.ITEM|OBTEST.WAREHOUSE|OBTEST.ORDERS|OBTEST.CUSTOMER|OBTEST.ORDER_LINE|OBTEST.NEW_ORDER|OBTEST.STOCK|OBTEST.HISTORY|&password=RM_oms&topic=ORACLE_np_59bb8o754l1c_59bbdrkasnc0-1-0&username=RM_oms&group=RM_oms : 559
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2637 64524,570406656] Find startpoint 1701148927 in store 0
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.DISTRICT' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.ITEM' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.WAREHOUSE' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.ORDERS' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.CUSTOMER' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.ORDER_LINE' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.NEW_ORDER' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.STOCK' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2838 64524,570406656] set fetch table name pattern 'OBTEST.HISTORY' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2890 64524,570406656] set fetch table name black pattern 'OBTEST.drc_txn*' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:2890 64524,570406656] set fetch table name black pattern 'OBTEST.DRC_TXN*' success.
2023-11-28 13:31:16 [NOTICE] [StoreManager.cpp:1036 64524,570406656] Get drcnet connection [0] from RM_oms:ORACLE_np_59bb8o754l1c_59bbdrkasnc0-1-0:10.186.16.126-17002-10.186.16.126-43444, groupid: 1, message type: sizedData
``` 

连接参数的配置位置: 

```
com.oceanbase.oms.store.client.impl.ClusterManagers#connectStore
``` 

writer.threshold已正确生效, 但实际效果并没有改观: 

```
watch com.oceanbase.oms.store.client.impl.ServerProxy getDrcNetResponse 'returnObj' 'returnObj.records.size()>1'
 
从DRC接到的数据仍然是一行一次
```
