---
title: 20231116 - Oracle性能测试 - 使用hammerdb
confluence_page_id: 2589482
created_at: 2023-11-16T08:48:23+00:00
updated_at: 2023-11-21T08:33:22+00:00
---

启动容器

```
 docker run --network=host -it --name hammerdb docker.io/tpcorg/hammerdb:oracle bash
``` 

安装Oracle sqlplus: 

```
wget https://download.oracle.com/otn_software/linux/instantclient/2111000/instantclient-sqlplus-linux.x64-21.11.0.0.0dbru.zip
 
解压到 /home/instantclient_21_5
``` 

配置tnsnames.ora: 

```
EE =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.186.16.126)(PORT = 1521))
 (CONNECT_DATA =
        (SERVICE_NAME = EE.ORACLE.DOCKER))
  )
``` 

测试连接: 

```
root@R740-26:/home/instantclient_21_5# /home/instantclient_21_5/sqlplus OBTEST/111111@EE

SQL*Plus: Release 21.0.0.0.0 - Production on Thu Nov 16 09:27:35 2023
Version 21.5.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> select count(1) from T1;

  COUNT(1)
----------
	 1
```
```
dbset db ora
dbset bm TPC-C
diset connection system_user obtest
diset connection system_password 111111
diset connection instance EE
diset tpcc tpcc_user obtest
diset tpcc tpcc_pass 111111

---
 

hammerdb>print dict
Dictionary Settings for Oracle
connection {
 system_user     = obtest
 system_password = 111111
 instance        = EE
 rac             = 0
}
tpcc       {
 count_ware       = 1
 num_vu           = 1
 tpcc_user        = obtest
 tpcc_pass        = 111111
 tpcc_def_tab     = users
 tpcc_ol_tab      = users
 tpcc_def_temp    = temp
 partition        = false
 hash_clusters    = false
 tpcc_tt_compat   = false
 total_iterations = 10000000
 raiseerror       = false
 keyandthink      = false
 checkpoint       = false
 ora_driver       = timed
 rampup           = 2
 duration         = 5
 allwarehouse     = false
 ora_timeprofile  = false
 async_scale      = false
 async_client     = 10
 async_verbose    = false
 async_delay      = 1000
 connect_pool     = false
}
``` 

载入数据: 

```
hammerdb>diset tpcc count_ware 10
Changed tpcc:count_ware from 1 to 10 for Oracle

hammerdb>diset tpcc num_vu 10
Changed tpcc:num_vu from 1 to 10 for Oracle

hammerdb>print dict
Dictionary Settings for Oracle
connection {
 system_user     = obtest
 system_password = 111111
 instance        = EE
 rac             = 0
}
tpcc       {
 count_ware       = 10
 num_vu           = 10
 tpcc_user        = obtest
 tpcc_pass        = 111111
 tpcc_def_tab     = users
 tpcc_ol_tab      = users
 tpcc_def_temp    = temp
 partition        = false
 hash_clusters    = false
 tpcc_tt_compat   = false
 total_iterations = 10000000
 raiseerror       = false
 keyandthink      = false
 checkpoint       = false
 ora_driver       = timed
 rampup           = 2
 duration         = 5
 allwarehouse     = false
 ora_timeprofile  = false
 async_scale      = false
 async_client     = 10
 async_verbose    = false
 async_delay      = 1000
 connect_pool     = false
}

hammerdb>buildschema
Script cleared
Building 10 Warehouses with 11 Virtual Users, 10 active + 1 Monitor VU(dict value num_vu is set to 10)
Ready to create a 10 Warehouse Oracle TPROC-C schema
in database ORACLE under user TPCC in tablespace USERS?
Enter yes or no: replied yes
Vuser 1 created - WAIT IDLE
Vuser 2 created - WAIT IDLE
Vuser 3 created - WAIT IDLE
Vuser 4 created - WAIT IDLE
Vuser 5 created - WAIT IDLE
Vuser 6 created - WAIT IDLE
Vuser 7 created - WAIT IDLE
Vuser 8 created - WAIT IDLE
Vuser 9 created - WAIT IDLE
Vuser 10 created - WAIT IDLE
Vuser 11 created - WAIT IDLE
Vuser 1:RUNNING
Vuser 1:Monitor Thread
Vuser 1:CREATING TPCC SCHEMA
Error in Virtual User 1: ORA-12154: TNS:could not resolve the connect identifier specified
Vuser 1:FINISHED FAILED
Vuser 2:RUNNING
Vuser 2:Worker Thread
Vuser 2:Waiting for Monitor Thread...
Vuser 3:RUNNING
Vuser 3:Worker Thread
Vuser 3:Waiting for Monitor Thread...
Vuser 4:RUNNING
Vuser 4:Worker Thread
Vuser 4:Waiting for Monitor Thread...
Vuser 5:RUNNING
``` 

或生成本地文件: 

```
hammerdb>print datagen
Data Generation set to build a TPC-C schema for Oracle with 1 warehouses with 1 virtual users in /tmp

hammerdb>dgset warehouse 10

hammerdb>dgset vu 10
Set virtual users to 10 for data generation

hammerdb>print datagen
Data Generation set to build a TPC-C schema for Oracle with 10 warehouses with 10 virtual users in /tmp

hammerdb>datagenrun
Ready to generate the data for a 10 Warehouse Oracle TPROC-C schema
in directory /tmp ?
Enter yes or no: replied yes
Vuser 1 created - WAIT IDLE
Vuser 2 created - WAIT IDLE
Vuser 3 created - WAIT IDLE
Vuser 4 created - WAIT IDLE
Vuser 5 created - WAIT IDLE
Vuser 6 created - WAIT IDLE
Vuser 7 created - WAIT IDLE
Vuser 8 created - WAIT IDLE
Vuser 9 created - WAIT IDLE
Vuser 10 created - WAIT IDLE
Vuser 11 created - WAIT IDLE
Vuser 1:RUNNING
Vuser 1:Monitor Thread
Vuser 1:Opened File /tmp/item_1.tbl
Vuser 1:Generating Item
Vuser 2:RUNNING
Vuser 2:Worker Thread
Vuser 2:Waiting for Monitor Thread...
Vuser 2:Generating 1 Warehouses start:1 end:1
Vuser 2:Start:Tue Nov 21 08:06:59 UTC 2023
Vuser 2:Opened File /tmp/warehouse_1.tbl
Vuser 2:Opened File /tmp/stock_1.tbl
Vuser 2:Opened File /tmp/district_1.tbl
Vuser 2:Opened File /tmp/customer_1.tbl
Vuser 2:Opened File /tmp/history_1.tbl
Vuser 2:Opened File /tmp/orders_1.tbl
Vuser 2:Opened File /tmp/new_order_1.tbl
Vuser 2:Opened File /tmp/order_line_1.tbl
Vuser 2:Generating Warehouse
Vuser 2:Generating Stock Wid=1
Vuser 3:RUNNING
Vuser 3:Worker Thread
Vuser 3:Waiting for Monitor Thread...
Vuser 3:Generating 1 Warehouses start:2 end:2
Vuser 3:Start:Tue Nov 21 08:07:00 UTC 2023
Vuser 3:Opened File /tmp/warehouse_2.tbl
Vuser 3:Opened File /tmp/stock_2.tbl
Vuser 3:Opened File /tmp/district_2.tbl
Vuser 3:Opened File /tmp/customer_2.tbl
Vuser 3:Opened File /tmp/history_2.tbl
Vuser 3:Opened File /tmp/orders_2.tbl
Vuser 3:Opened File /tmp/new_order_2.tbl
Vuser 3:Opened File /tmp/order_line_2.tbl
Vuser 3:Generating Warehouse
Vuser 3:Generating Stock Wid=2
Vuser 4:RUNNING
Vuser 4:Worker Thread
Vuser 4:Waiting for Monitor Thread...
Vuser 4:Generating 1 Warehouses start:3 end:3
Vuser 4:Start:Tue Nov 21 08:07:00 UTC 2023
Vuser 4:Opened File /tmp/warehouse_3.tbl
Vuser 4:Opened File /tmp/stock_3.tbl
Vuser 4:Opened File /tmp/district_3.tbl
Vuser 4:Opened File /tmp/customer_3.tbl
Vuser 4:Opened File /tmp/history_3.tbl
Vuser 4:Opened File /tmp/orders_3.tbl
Vuser 4:Opened File /tmp/new_order_3.tbl
Vuser 4:Opened File /tmp/order_line_3.tbl
Vuser 4:Generating Warehouse
Vuser 4:Generating Stock Wid=3
Vuser 5:RUNNING
Vuser 5:Worker Thread
Vuser 5:Waiting for Monitor Thread...
Vuser 5:Generating 1 Warehouses start:4 end:4
Vuser 5:Start:Tue Nov 21 08:07:01 UTC 2023
Vuser 5:Opened File /tmp/warehouse_4.tbl
Vuser 5:Opened File /tmp/stock_4.tbl
Vuser 5:Opened File /tmp/district_4.tbl
Vuser 5:Opened File /tmp/customer_4.tbl
Vuser 5:Opened File /tmp/history_4.tbl
Vuser 5:Opened File /tmp/orders_4.tbl
Vuser 5:Opened File /tmp/new_order_4.tbl
Vuser 5:Opened File /tmp/order_line_4.tbl
Vuser 5:Generating Warehouse
Vuser 5:Generating Stock Wid=4
Vuser 6:RUNNING
``` 

设置vu数量: 

```
vuset vu 2
``` 

进行TPCC测试: 

```
hammerdb>vurun
Script loaded, Type "print script" to view
Vuser 1 created MONITOR - WAIT IDLE
Vuser 2 created - WAIT IDLE
Vuser 3 created - WAIT IDLE
3 Virtual Users Created with Monitor VU
Vuser 1:RUNNING
Vuser 1:Beginning rampup time of 2 minutes
Vuser 2:RUNNING
Vuser 2:Processing 10000000 transactions with output suppressed...
Vuser 3:RUNNING
Vuser 3:Processing 10000000 transactions with output suppressed...
Vuser 1:Rampup 1 minutes complete ...
Vuser 1:Rampup 2 minutes complete ...
Vuser 1:Rampup complete, Taking start AWR snapshot.
Vuser 1:Start Snapshot 93 taken at 17 NOV 2023 02:27 of instance EE (1) of database EE (2066100034)
Vuser 1:Timing test period of 5 in minutes
Vuser 1:1 ...,
Vuser 1:2 ...,
Vuser 1:3 ...,
Vuser 1:4 ...,
Vuser 1:5 ...,
Vuser 1:Test complete, Taking end AWR snapshot.
Benchmark Run jobid=6556CF0D60A403E213331363
hammerdb>Vuser 1:End Snapshot 94 taken at 17 NOV 2023 02:32 of instance EE (1) of database EE (2066100034)
Vuser 1:Test complete: view report from SNAPID 93 to 94
Vuser 1:2 Active Virtual Users configured
Vuser 1:TEST RESULT : System achieved 8043 NOPM from 16365 Oracle TPM
Vuser 1:FINISHED SUCCESS
Vuser 3:FINISHED SUCCESS
Vuser 2:FINISHED SUCCESS
ALL VIRTUAL USERS COMPLETE
``` 

# OMS全量迁移的bug (TODO)

使用OMS全量迁移时, 会出现报错: 

```
[2023-11-16 19:26:42.232] [ERROR] [sinkTask-3] [Fatal exception oms record [OmsRecord{prevStruct=null, postStruct=Struct{NO_W_ID(NUMBER)=1,NO_D_ID(NUMBER)=1,N
O_O_ID(NUMBER)=2302}, recordType=ROW, meta={OmsMeta}:dbType:ORACLE,db:OBTEST,table:NEW_ORDER,checkpoint:null,timestamp:null.null,sourceIdentity:jdbc:oracle:th
in:@//10.186.16.126:1521/EE.ORACLE.DOCKER,sourceDB:OBTEST,sourceTable:NEW_ORDER,transactionId:null,uniqueId:null,traceId:null,shardInfo:shardColumns:[null], s
inkRoute:null, shardDB:OBTEST, shardTB:NEW_ORDER}], exception message [{}]]
com.oceanbase.oms.record.exception.DataException: rowid is not a valid field name
        at com.oceanbase.oms.record.oms.OmsStruct.lookupField(OmsStruct.java:179)
        at com.oceanbase.oms.record.oms.OmsStruct.getFieldValue(OmsStruct.java:78)
        at com.oceanbase.oms.connector.common.util.RowIDUtil.generateColumnInfo(RowIDUtil.java:66)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.getFieldValuesPlaceholder(AbstractPrepareStatementBuilder.java:266)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.buildInsert(AbstractPrepareStatementBuilder.java:92)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.build(AbstractPrepareStatementBuilder.java:53)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.build(AbstractPrepareStatementBuilder.java:33)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.realWriteDML(Writer.java:541)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.writeDML(Writer.java:598)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.executeInRetry(Writer.java:480)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushDML(Writer.java:391)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.batchFlushDML(Writer.java:287)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushFullInsertBatch(Writer.java:230)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushRecords(Writer.java:195)
        at com.oceanbase.oms.connector.jdbc.sink.DefaultJDBCSink.offer(DefaultJDBCSink.java:60)
        at com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer(OBOracleJDBCSink.java:25)
        at com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask.run(SyncSinkConnectorTask.java:44)
        at java.lang.Thread.run(Thread.java:853)
``` 

将源端表清空(truncate), 在全量迁移后, 在增量迁移过程中再写入数据, 则可以通过

```
EE =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.186.16.126)(PORT = 1521))
 (CONNECT_DATA =
        (SERVICE_NAME = EE.ORACLE.DOCKER))
  )
``` 

测试连接: 

```
root@R740-26:/home/instantclient_21_5# /home/instantclient_21_5/sqlplus OBTEST/111111@EE

SQL*Plus: Release 21.0.0.0.0 - Production on Thu Nov 16 09:27:35 2023
Version 21.5.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> select count(1) from T1;

  COUNT(1)
----------
	 1
```

```
dbset db ora
dbset bm TPC-C
diset connection system_user obtest
diset connection system_password 111111
diset connection instance EE
diset tpcc tpcc_user obtest
diset tpcc tpcc_pass 111111

---
 

hammerdb>print dict
Dictionary Settings for Oracle
connection {
 system_user     = obtest
 system_password = 111111
 instance        = EE
 rac             = 0
}
tpcc       {
 count_ware       = 1
 num_vu           = 1
 tpcc_user        = obtest
 tpcc_pass        = 111111
 tpcc_def_tab     = users
 tpcc_ol_tab      = users
 tpcc_def_temp    = temp
 partition        = false
 hash_clusters    = false
 tpcc_tt_compat   = false
 total_iterations = 10000000
 raiseerror       = false
 keyandthink      = false
 checkpoint       = false
 ora_driver       = timed
 rampup           = 2
 duration         = 5
 allwarehouse     = false
 ora_timeprofile  = false
 async_scale      = false
 async_client     = 10
 async_verbose    = false
 async_delay      = 1000
 connect_pool     = false
}
``` 

载入数据: 

```
hammerdb>buildschema
Script cleared
Building 1 Warehouses(s) with 1 Virtual User
Ready to create a 1 Warehouse Oracle TPROC-C schema
in database EE under user OBTEST in tablespace USERS?
Enter yes or no: replied yes
Vuser 1 created - WAIT IDLE
Vuser 1:RUNNING
Vuser 1:CREATING OBTEST SCHEMA
Vuser 1:CREATING USER obtest
Vuser 1:ORA-01920: user name 'OBTEST' conflicts with another user or role name create user obtest identified by 111111 default tablespace users temporary tablespace temp
Vuser 1:CREATING TPCC TABLES
Vuser 1:Loading Item
Vuser 1:Loading Items - 50000
Vuser 1:Loading Items - 100000
Vuser 1:Item done
Vuser 1:Start:Thu Nov 16 09:33:27 UTC 2023
Vuser 1:Loading Warehouse
Vuser 1:Loading Stock Wid=1
Vuser 1:Loading Stock - 50000
Vuser 1:Loading Stock - 100000
Vuser 1:Stock done
Vuser 1:Loading District
Vuser 1:District done
Vuser 1:Loading Customer for DID=1 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=2 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=3 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=4 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=5 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=6 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=7 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=8 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=9 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Customer for DID=10 WID=1
Vuser 1:Customer Done
Vuser 1:Loading Orders for D=1 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=2 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=3 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=4 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=5 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=6 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=7 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=8 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=9 W=1
Vuser 1:Orders Done
Vuser 1:Loading Orders for D=10 W=1
Vuser 1:Orders Done
Vuser 1:End:Thu Nov 16 09:34:33 UTC 2023
Vuser 1:CREATING TPCC INDEXES
Vuser 1:CREATING TPCC STORED PROCEDURES
Vuser 1:GATHERING SCHEMA STATISTICS
Vuser 1:OBTEST SCHEMA COMPLETE
Vuser 1:FINISHED SUCCESS
ALL VIRTUAL USERS COMPLETE
Schema Build jobid=6555E1C860A403E253030383
``` 

使用OMS迁移时, 会出现报错: 

```
[2023-11-16 19:26:42.227] [INFO] [sinkTask-0] [batch replicate sql failed, use single statement replicate]
com.oceanbase.oms.record.exception.DataException: rowid is not a valid field name
        at com.oceanbase.oms.record.oms.OmsStruct.lookupField(OmsStruct.java:179)
        at com.oceanbase.oms.record.oms.OmsStruct.getFieldValue(OmsStruct.java:78)
        at com.oceanbase.oms.connector.common.util.RowIDUtil.generateColumnInfo(RowIDUtil.java:66)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.getFieldValuesPlaceholder(AbstractPrepareStatementBuilder.java:266)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.buildInsert(AbstractPrepareStatementBuilder.java:92)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.build(AbstractPrepareStatementBuilder.java:53)
        at com.oceanbase.oms.connector.jdbc.sink.statement.AbstractPrepareStatementBuilder.build(AbstractPrepareStatementBuilder.java:33)
        at com.oceanbase.oms.connector.jdbc.sink.batch.SemiPrepareStatementBatchAccumulate.buildPrepareStatement(SemiPrepareStatementBatchAccumulate.java:56)
        at com.oceanbase.oms.connector.jdbc.sink.batch.SemiPrepareStatementBatchAccumulate.tryDrainBatch(SemiPrepareStatementBatchAccumulate.java:42)
        at com.oceanbase.oms.connector.jdbc.sink.batch.SemiPrepareStatementBatchAccumulate.tryDrainBatch(SemiPrepareStatementBatchAccumulate.java:25)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.tryBatchRecord(Writer.java:462)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.batchFlushDML(Writer.java:253)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushFullInsertBatch(Writer.java:230)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushRecords(Writer.java:195)
        at com.oceanbase.oms.connector.jdbc.sink.DefaultJDBCSink.offer(DefaultJDBCSink.java:60)
        at com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer(OBOracleJDBCSink.java:25)
        at com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask.run(SyncSinkConnectorTask.java:44)
        at java.lang.Thread.run(Thread.java:853)
``` 

# hammerDB hang

如果使用truncate清理了源端数据, 需要使用 buildschema 进行初始数据的load. 

如果直接进行vurun, 那么会hang. hang的原因是没有warehouse, 所以order id的计算随机值就陷入了死循环

# hammerDB 调试脚本

  - print script, 可以将配置对应的压力脚本 (TCL脚本) 打印出来
  - customscript {script}, 可以将自定义的压力脚本载入
