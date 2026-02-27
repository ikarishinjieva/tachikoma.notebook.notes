---
title: 20231123 - 在Oracle并发压力中, 测试OMS性能 [3]
confluence_page_id: 2589678
created_at: 2023-11-23T07:11:06+00:00
updated_at: 2023-11-27T01:51:02+00:00
---

# 背景

  - 场景: 1 warehouse, 1 vu, 仅做buildschema数据初始化, 数据同步慢, 进行如下调优: 
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

# 调优后测试

vu=1, warehouse=1, buildschema压力, 整体延迟在10+s

通过目标端的Sink并发度 和 SQL并发度观察: 

```
thread | grep sink | grep RUN | wc -l

vmtool --action getInstances --className com.oceanbase.jdbc.internal.protocol.AbstractConnectProtocol --limit 10000 --express '
{ "ObConnectionRunning=" + instances.{? #this.lock.isLocked()}.size()}
'
``` 

回放数据时, 可以达到接近60, 可以提高并发阈值来换取延迟: 

将目标端 sink.workerNum = 128

可以看到SQL并发度能达到100左右

在源端大事务后, 会有几秒的卡顿

测试卡顿期间, 源端的阻塞状况: 

```
 ["1: LogEntryFetcher_OutQueue_Size=0","2: OracleLogExtractor_OutQueue_Size=1631","3: LogRecordSerializer_OutQueue_Size=0","4: LogRecordConverterPrepare_OutQueue_Size=9993","5: LogRecordConverterCore_OutQueue_Size=8","6: RecordDispatcher_OutQueue_Ids_Size=0","6: RecordDispatcher_OutQueue_Size=0","6/7.1: RecordDispatcher_OutQueue_Records_Size=8","7.2: global: scnIncrementRecordQueue_Size=12"]

["1: LogEntryFetcher_OutQueue_Size=0","2: OracleLogExtractor_OutQueue_Size=9433","3: LogRecordSerializer_OutQueue_Size=6","4: LogRecordConverterPrepare_OutQueue_Size=3010","5: LogRecordConverterCore_OutQueue_Size=1","6: RecordDispatcher_OutQueue_Ids_Size=0","6: RecordDispatcher_OutQueue_Size=0","6/7.1: RecordDispatcher_OutQueue_Records_Size=0","7.2: global: scnIncrementRecordQueue_Size=0"]
 
["1: LogEntryFetcher_OutQueue_Size=168","2: OracleLogExtractor_OutQueue_Size=3896","3: LogRecordSerializer_OutQueue_Size=0","4: LogRecordConverterPrepare_OutQueue_Size=4","5: LogRecordConverterCore_OutQueue_Size=0","6: RecordDispatcher_OutQueue_Ids_Size=0","6: RecordDispatcher_OutQueue_Size=0","6/7.1: RecordDispatcher_OutQueue_Records_Size=0","7.2: global: scnIncrementRecordQueue_Size=0"]
``` 

看起来处理器5和处理器3会在某种情况下比较慢

先诊断处理器5, 其并发数配置为 logminer.converter_thread_num, 默认值8

调大配置: deliver2store.logminer.converter_thread_num = 16

观察到队列4的阻塞情况缓解, 阻塞会集中于队列2

开始诊断队列2的积压, 查看处理器3的火焰图: 

![image2023-11-26 23:36:11.png](/assets/01KJBZ428QFGM47RNAB68EERFS/image2023-11-26%2023%3A36%3A11.png)

主要的CPU消耗在大事务在磁盘的存储处理上: 

  - 写入磁盘: 在遇到commit前, 在处理器3中暂存数据到文件
  - 读取磁盘: 在遇到commit后, 在处理器3中读取数据

问题: 在同一个处理器中, 进行数据 往文件暂存时, 如何降低序列化/反序列化的成本? (FileStore有缓存结构, 但缓存的是序列化后的文本, 使用缓存时, 需要反序列化)

# 技巧: 判断输入端的事务的大小

```
watch com.taobao.drc.logminer.aggregator.OracleLogKVStoreAggregator dealCommittedTransaction '{ @com.taobao.drc.logminer.dictionary.OracleDictionaries@getTable(params[0].bigListRecords.get(1).table) == null ? null : @com.taobao.drc.logminer.dictionary.OracleDictionaries@getTable(params[0].bigListRecords.get(1).table).table(), params[0].recordIDs.size()}' 'params[0].recordIDs.size()>2' -n 100000
```
