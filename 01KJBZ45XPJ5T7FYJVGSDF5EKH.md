---
title: 20231128 - 整理OMS的全量复制链路 的过程
confluence_page_id: 2589765
created_at: 2023-11-28T10:04:51+00:00
updated_at: 2023-12-04T08:31:14+00:00
---

# 目录

# 进程

  1. connector
     1. 命令行

```
ds       19483 67.1  1.5 19017760 4018300 pts/0 Sl  16:28   3:03 ./java -server -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/ds/run/10.186.16.126-9000:connector_v2:np_59s0ihb35ekg-full_trans-1-0:0008000001 -server -Xms8g -Xmx8g -Xmn4g -Xss512k -Djava.library.path=/home/ds/plugins/jdbc_connector/ -Duser.home=/home/ds -Dlogging.path=/home/ds/run/10.186.16.126-9000:connector_v2:np_59s0ihb35ekg-full_trans-1-0:0008000001/logs -Djavax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl -cp jdbc_connector.jar:connector-dataflow.jar:jdbc-sink-ob-oracle.jar:conf com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap -c conf -t 10.186.16.126-9000:connector_v2:np_59s0ihb35ekg-full_trans-1-0:0008000001 start
``` 
     2. 主目录: /u01/ds/run/10.186.16.126-9000:connector_v2:np_59s0ihb35ekg-full_trans-1-0:0008000001

# 火焰图

命令

```
[root@R740-26 async-profiler-2.9-linux-x64]# ./profiler.sh -t start 32596 --fdtransfer -o flamegraph
[root@R740-26 async-profiler-2.9-linux-x64]# ./profiler.sh -t stop 32596 --fdtransfer -o flamegraph
``` 

图: [1.html](/assets/01KJBZ45XPJ5T7FYJVGSDF5EKH/1.html)

简要分析: 

  - 活跃的线程是两种: sourceTask, sinkTask
  - 需要分析: 
    - 线程的管理结构
    - 数据的流向

# 主启动器

com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap

启动配置: 

```
[root@R740-26 conf]# pwd
/u01/ds/run/10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002/conf
[root@R740-26 conf]# grep "" *
checkpoint.chk:{
checkpoint.chk:  "version" : 1,
checkpoint.chk:  "tableItems" : [ {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.ORDER_LINE",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 224,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76327,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 194304,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.STOCK",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 360,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76329,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 65408,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.ORDERS",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 28,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76325,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 181888,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.DISTRICT",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 15,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76317,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 67976,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.WAREHOUSE",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 3,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76331,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 19488,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.CUSTOMER",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 216,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76315,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 97664,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.ITEM",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 25,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76320,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 42240,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.HISTORY",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 33,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76318,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 94336,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  }, {
checkpoint.chk:    "clientId" : "jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
checkpoint.chk:    "fullTableName" : "OBTEST.NEW_ORDER",
checkpoint.chk:    "partitionInfo" : null,
checkpoint.chk:    "sliceItems" : [ {
checkpoint.chk:      "id" : 18,
checkpoint.chk:      "splitType" : "BLOCK",
checkpoint.chk:      "bottom" : {
checkpoint.chk:        "OMS_OBJECT_NUMBER" : 76322,
checkpoint.chk:        "OMS_RELATIVE_FNO" : 5,
checkpoint.chk:        "OMS_BLOCK_NUMBER" : 176768,
checkpoint.chk:        "OMS_ROW_NUMBER" : 0
checkpoint.chk:      },
checkpoint.chk:      "top" : null,
checkpoint.chk:      "success" : true,
checkpoint.chk:      "last" : true
checkpoint.chk:    } ]
checkpoint.chk:  } ]
checkpoint.chk:}
condition.json:{
condition.json:	"whiteCondition":[
condition.json:		{
condition.json:			"all":false,
condition.json:			"sub":[
condition.json:				{
condition.json:					"name":"CUSTOMER",
condition.json:					"map":"CUSTOMER"
condition.json:				},
condition.json:				{
condition.json:					"name":"DISTRICT",
condition.json:					"map":"DISTRICT"
condition.json:				},
condition.json:				{
condition.json:					"name":"HISTORY",
condition.json:					"map":"HISTORY"
condition.json:				},
condition.json:				{
condition.json:					"name":"ITEM",
condition.json:					"map":"ITEM"
condition.json:				},
condition.json:				{
condition.json:					"name":"NEW_ORDER",
condition.json:					"map":"NEW_ORDER"
condition.json:				},
condition.json:				{
condition.json:					"name":"ORDERS",
condition.json:					"map":"ORDERS"
condition.json:				},
condition.json:				{
condition.json:					"name":"ORDER_LINE",
condition.json:					"map":"ORDER_LINE"
condition.json:				},
condition.json:				{
condition.json:					"name":"STOCK",
condition.json:					"map":"STOCK"
condition.json:				},
condition.json:				{
condition.json:					"name":"WAREHOUSE",
condition.json:					"map":"WAREHOUSE"
condition.json:				}
condition.json:			],
condition.json:			"name":"OBTEST",
condition.json:			"map":"OBTEST"
condition.json:		}
condition.json:	]
condition.json:}
coordinator.json:{
coordinator.json:	"enableMetricReportTask":"false",
coordinator.json:	"taskIdentity":"np_59s1c3qy1kkg",
coordinator.json:	"enableOmsConnectorV2Report":"true",
coordinator.json:	"customizeClass":"com.oceanbase.oms.connector.fullcustomize.FullFrameworkCustomize",
coordinator.json:	"sourceType":"ORACLE",
coordinator.json:	"timezone":"+08:00",
coordinator.json:	"enableActiveReportTask":"false",
coordinator.json:	"taskSubId":"1",
coordinator.json:	"checkDstTableEmpty":"false",
coordinator.json:	"sinkType":"OB_ORACLE",
coordinator.json:	"listenPort":"16001",
coordinator.json:	"connectorJvmParam":"-server -Xms8g -Xmx8g -Xmn4g -Xss512k"
coordinator.json:}
runningConf.json:{
runningConf.json:  "condition" : {
runningConf.json:    "whiteCondition" : "[{\"all\":false,\"sub\":[{\"name\":\"CUSTOMER\",\"map\":\"CUSTOMER\"},{\"name\":\"DISTRICT\",\"map\":\"DISTRICT\"},{\"name\":\"HISTORY\",\"map\":\"HISTORY\"},{\"name\":\"ITEM\",\"map\":\"ITEM\"},{\"name\":\"NEW_ORDER\",\"map\":\"NEW_ORDER\"},{\"name\":\"ORDERS\",\"map\":\"ORDERS\"},{\"name\":\"ORDER_LINE\",\"map\":\"ORDER_LINE\"},{\"name\":\"STOCK\",\"map\":\"STOCK\"},{\"name\":\"WAREHOUSE\",\"map\":\"WAREHOUSE\"}],\"name\":\"OBTEST\",\"map\":\"OBTEST\"}]"
runningConf.json:  },
runningConf.json:  "coordinator" : {
runningConf.json:    "caseStrategy" : "follow-source",
runningConf.json:    "taskIdentity" : "np_59s1c3qy1kkg",
runningConf.json:    "enableOmsConnectorV2Report" : "true",
runningConf.json:    "timezone" : "+08:00",
runningConf.json:    "enableActiveReportTask" : "false",
runningConf.json:    "taskSubId" : "1",
runningConf.json:    "checkDstTableEmpty" : "false",
runningConf.json:    "bridgeQueueSize" : "128",
runningConf.json:    "enableMetricReportTask" : "false",
runningConf.json:    "customizeClass" : "com.oceanbase.oms.connector.fullcustomize.FullFrameworkCustomize",
runningConf.json:    "sourceType" : "ORACLE",
runningConf.json:    "throttleMemoryBound" : "2147483648",
runningConf.json:    "printTraceMsg" : "false",
runningConf.json:    "sinkType" : "OB_ORACLE",
runningConf.json:    "taskResume" : "false",
runningConf.json:    "listenPort" : "16001",
runningConf.json:    "ignoreCompensateDDL" : "false",
runningConf.json:    "connectorJvmParam" : "-server -Xms8g -Xmx8g -Xmn4g -Xss512k"
runningConf.json:  },
runningConf.json:  "sink" : {
runningConf.json:    "boosterClass" : "com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSinkBooster",
runningConf.json:    "caseStrategy" : "follow-source",
runningConf.json:    "enableNoUniqueConstraintTableReplicate" : "false",
runningConf.json:    "accurateInvalidateCache" : "true",
runningConf.json:    "localRegionNo" : "2147473648",
runningConf.json:    "timezone" : "+08:00",
runningConf.json:    "dbVersion" : "4.2.0.0",
runningConf.json:    "filterHiddenPK" : "false",
runningConf.json:    "isPreLoadAllTableSchema" : "false",
runningConf.json:    "writeMode" : "full",
runningConf.json:    "type" : "JDBC_SINK_OB_ORACLE",
runningConf.json:    "useSourceRecord" : "true",
runningConf.json:    "password" : "111111",
runningConf.json:    "canLobBatched" : "false",
runningConf.json:    "jdbcUrl" : "jdbc:oceanbase://10.186.16.126:32883?useUnicode=true&allowMultiQueries=true&socketTimeout=50000&characterEncoding=utf8&sendConnectionAttributes=false",
runningConf.json:    "sinkMaps" : "[[\"OBTEST\",\"HISTORY\"],[\"OBTEST\",\"STOCK\"],[\"OBTEST\",\"CUSTOMER\"],[\"OBTEST\",\"NEW_ORDER\"],[\"OBTEST\",\"ORDER_LINE\"],[\"OBTEST\",\"WAREHOUSE\"],[\"OBTEST\",\"ORDERS\"],[\"OBTEST\",\"DISTRICT\"],[\"OBTEST\",\"ITEM\"]]",
runningConf.json:    "jar" : "jdbc-sink-ob-oracle.jar",
runningConf.json:    "workerNum" : "8",
runningConf.json:    "sinkType" : "OB_ORACLE",
runningConf.json:    "supportedTimezone" : "[]",
runningConf.json:    "forceUpdateTimeField" : "false",
runningConf.json:    "username" : "OBTEST@oms_oracle#obthree"
runningConf.json:  },
runningConf.json:  "source" : {
runningConf.json:    "boosterClass" : "com.oceanbase.oms.connector.source.dataflow.DataFlowSourceBooster",
runningConf.json:    "enableOmsConnectorV2Report" : "true",
runningConf.json:    "clients" : "[{\"password\":\"111111\",\"clientId\":\"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER\",\"instance\":\"10.186.16.126:1521/EE.ORACLE.DOCKER\",\"username\":\"OBTEST\"}]",
runningConf.json:    "sliceBatchSize" : "600",
runningConf.json:    "sliceWorkerNum" : "8",
runningConf.json:    "sourceJdbcTableCache" : "true",
runningConf.json:    "timezone" : "+08:00",
runningConf.json:    "dbType" : "ORACLE",
runningConf.json:    "databaseMaxConnection" : "50",
runningConf.json:    "type" : "DATAFLOW_SOURCE",
runningConf.json:    "sourceBatchMemorySize" : "16777216",
runningConf.json:    "filterTmpTable" : "true",
runningConf.json:    "jar" : "connector-dataflow.jar",
runningConf.json:    "workerNum" : "8",
runningConf.json:    "taskResume" : "false",
runningConf.json:    "batchSize" : "512"
runningConf.json:  }
runningConf.json:}
sink.json:{
sink.json:	"boosterClass":"com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSinkBooster",
sink.json:	"password":"111111",
sink.json:	"enableNoUniqueConstraintTableReplicate":"false",
sink.json:	"timezone":"+08:00",
sink.json:	"jdbcUrl":"jdbc:oceanbase://10.186.16.126:32883?useUnicode=true&allowMultiQueries=true&socketTimeout=50000&characterEncoding=utf8",
sink.json:	"jar":"jdbc-sink-ob-oracle.jar",
sink.json:	"isPreLoadAllTableSchema":"false",
sink.json:	"workerNum":"8",
sink.json:	"type":"JDBC_SINK_OB_ORACLE",
sink.json:	"username":"OBTEST@oms_oracle#obthree"
sink.json:}
source.json:{
source.json:	"boosterClass":"com.oceanbase.oms.connector.source.dataflow.DataFlowSourceBooster",
source.json:	"clients":[
source.json:		{
source.json:			"password":"111111",
source.json:			"clientId":"jdbc:oracle:thin:@//10.186.16.126:1521/EE.ORACLE.DOCKER",
source.json:			"instance":"10.186.16.126:1521/EE.ORACLE.DOCKER",
source.json:			"username":"OBTEST"
source.json:		}
source.json:	],
source.json:	"sliceBatchSize":"600",
source.json:	"sliceWorkerNum":"8",
source.json:	"timezone":"+08:00",
source.json:	"filterTmpTable":"true",
source.json:	"dbType":"ORACLE",
source.json:	"databaseMaxConnection":"50",
source.json:	"jar":"connector-dataflow.jar",
source.json:	"workerNum":"8",
source.json:	"taskResume":"false",
source.json:	"type":"DATAFLOW_SOURCE"
source.json:}
[root@R740-26 conf]#
``` 

# 通过发生SQL堆栈, 分析机制

```
命令: 
java -jar /opt/arthas/arthas-boot.jar --telnet-port 9998 --http-port -1 -c 'stack java.sql.Connection prepareStatement -n 1000' $(ps aux | grep java | grep coordinator | awk '{print $2}')
``` 

结果: 

### 堆栈1:

```
堆栈1: 
ts=2023-11-28 23:53:47;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.jdbc.OceanBaseConnection.prepareStatement()
        at com.alibaba.druid.pool.DruidPooledConnection.prepareStatement(DruidPooledConnection.java:377)
        at com.oceanbase.oms.dataflow.jdbcclient.JdbcUtils.queryTemplate(JdbcUtils.java:126)
        at com.oceanbase.oms.dataflow.jdbcclient.AbstractJDBCClient.getGlobalConfigs(AbstractJDBCClient.java:1398)
        at com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor.setConfigByDBConfig(JDBCPreprocessor.java:1065)
        at com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor.setConfigByDBConfig(JDBCPreprocessor.java:1058)
        at com.oceanbase.oms.connector.fullcustomize.preprocessor.JDBCPreprocessor.preprocess(JDBCPreprocessor.java:200)
        at com.oceanbase.oms.connector.fullcustomize.FullFrameworkCustomize.coordinate(FullFrameworkCustomize.java:86)
        at com.oceanbase.oms.connector.coordinator.BootStrapPanel.boot(BootStrapPanel.java:168)
        at com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap.main(Bootstrap.java:21)
 
``` 

获取数据库的配置参数. SQL: "SELECT PARAMETER,VALUE FROM SYS.NLS_DATABASE_PARAMETERS WHERE PARAMETER = %s"

遗漏的机制: 全量复制的定制器

### 堆栈2: 

```
---
堆栈2: 
ts=2023-11-28 23:53:48;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.jdbc.OceanBaseConnection.prepareStatement()
        at com.alibaba.druid.pool.DruidPooledConnection.prepareStatement(DruidPooledConnection.java:377)
        at com.oceanbase.oms.dataflow.jdbcclient.AbstractJDBCClient.getDBVersion(AbstractJDBCClient.java:1174)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oboracle.OBOracleJDBCClient.getDBVersion(OBOracleJDBCClient.java:677)
        at com.oceanbase.oms.connector.cache.AbstractTableSchemaCache.getDbVersion(AbstractTableSchemaCache.java:308)
        at com.oceanbase.oms.connector.cache.TableSchemaCacheManager.getDbVersion(TableSchemaCacheManager.java:34)
        at com.oceanbase.oms.connector.jdbc.sink.AbstractJDBCSinkBooster.addVersionToSinkContext(AbstractJDBCSinkBooster.java:126)
        at com.oceanbase.oms.connector.jdbc.sink.AbstractJDBCSinkBooster.initSinkEnv(AbstractJDBCSinkBooster.java:85)
        at com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSinkBooster.initSinkEnv(OBOracleJDBCSinkBooster.java:39)
        at com.oceanbase.oms.connector.coordinator.BootStrapPanel.boot(BootStrapPanel.java:182)
        at com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap.main(Bootstrap.java:21)
``` 

使用目标数据库的配置, 初始化Sink的配置 (时区和数据库版本)

### 堆栈3:

```
 
---
堆栈3:
ts=2023-11-28 23:53:49;thread_name=slice-worker-7;id=40;is_daemon=true;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @oracle.jdbc.driver.PhysicalConnection.prepareStatement()
        at oracle.jdbc.driver.PhysicalConnection.prepareStatement(PhysicalConnection.java:1755)
        at oracle.jdbc.driver.PhysicalConnection.prepareStatement(PhysicalConnection.java:1663)
        at com.alibaba.druid.pool.DruidPooledConnection.prepareStatement(DruidPooledConnection.java:377)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient.blockSlice(OracleJDBCClient.java:443)
        at com.oceanbase.oms.dataflow.slice.BlockSliceService.lambda$slice$2(BlockSliceService.java:64)
        at com.oceanbase.oms.connector.common.util.RetryUtil.retryTask(RetryUtil.java:33)
        at com.oceanbase.oms.dataflow.slice.BlockSliceService.slice(BlockSliceService.java:59)
        at com.oceanbase.oms.dataflow.slice.LocalSliceProvider.provider(LocalSliceProvider.java:58)
        at com.oceanbase.oms.connector.source.dataflow.DataFlowSliceTask.doSlice(DataFlowSliceTask.java:67)
        at com.oceanbase.oms.connector.source.dataflow.SliceTaskManager$InnerSliceTask.run(SliceTaskManager.java:510)
        at java.lang.Thread.run(Thread.java:853)
``` 

已补充 数据切分器逻辑

### 堆栈4: 

```
---
 
堆栈4: 
 
ts=2023-11-28 23:53:49;thread_name=sourceTask-6;id=55;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @oracle.jdbc.driver.PhysicalConnection.prepareStatement()
        at oracle.jdbc.driver.PhysicalConnection.prepareStatement(PhysicalConnection.java:1755)
        at oracle.jdbc.driver.PhysicalConnection.prepareStatement(PhysicalConnection.java:1663)
        at com.alibaba.druid.pool.DruidPooledConnection.prepareStatement(DruidPooledConnection.java:377)
        at com.oceanbase.oms.dataflow.jdbcclient.AbstractQueryClient.queryDataBySlice(AbstractQueryClient.java:762)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleQueryClient.queryDataBySliceBlock(OracleQueryClient.java:485)
        at com.oceanbase.oms.dataflow.jdbcclient.AbstractJDBCClient.queryDataBySlice(AbstractJDBCClient.java:1233)
        at com.oceanbase.oms.connector.source.dataflow.DataFlowSource.lambda$poll$0(DataFlowSource.java:183)
        at com.oceanbase.oms.connector.common.util.RetryUtil.retryTask(RetryUtil.java:33)
        at com.oceanbase.oms.connector.source.dataflow.DataFlowSource.poll(DataFlowSource.java:169)
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.pollQueryConditionWithRetry(ConditionedSourceConnectorTask.java:106)
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.run(ConditionedSourceConnectorTask.java:68)
        at java.lang.Thread.run(Thread.java:853)
``` 

已补充数据获取逻辑

### 堆栈5 + 6:

```
---
堆栈5: 
ts=2023-11-28 23:53:50;thread_name=sinkTask-3;id=49;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.jdbc.OceanBaseConnection.prepareStatement()
        at com.alibaba.druid.pool.DruidPooledConnection.prepareStatement(DruidPooledConnection.java:377)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient.getTablesInner(AbstractOracleJDBCClient.java:527)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oboracle.OBOracleJDBCClient.getTables(OBOracleJDBCClient.java:409)
        at com.oceanbase.oms.connector.cache.AbstractTableSchemaCache.loadTableInfoFromDb(AbstractTableSchemaCache.java:189)
        at com.oceanbase.oms.connector.cache.AbstractTableSchemaCache.getSchema(AbstractTableSchemaCache.java:129)
        at com.oceanbase.oms.connector.cache.TableSchemaCacheManager.getSchema(TableSchemaCacheManager.java:27)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.tryBatchRecord(Writer.java:461)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.batchFlushDML(Writer.java:253)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushFullInsertBatch(Writer.java:230)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushRecords(Writer.java:195)
        at com.oceanbase.oms.connector.jdbc.sink.DefaultJDBCSink.offer(DefaultJDBCSink.java:60)
        at com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer(OBOracleJDBCSink.java:25)
        at com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask.run(SyncSinkConnectorTask.java:44)
        at java.lang.Thread.run(Thread.java:853)
 
 
堆栈6: 
ts=2023-11-28 23:53:50;thread_name=sinkTask-3;id=49;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.jdbc.OceanBaseConnection.prepareStatement()
        at com.alibaba.druid.pool.DruidPooledConnection.prepareStatement(DruidPooledConnection.java:377)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient.getTableColumnNames(AbstractOracleJDBCClient.java:303)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient.initTableColumns(AbstractOracleJDBCClient.java:238)
        at com.oceanbase.oms.dataflow.jdbcclient.AbstractJDBCClient.afterTableLoad(AbstractJDBCClient.java:1195)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.AbstractOracleJDBCClient.getTablesInner(AbstractOracleJDBCClient.java:572)
        at com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oboracle.OBOracleJDBCClient.getTables(OBOracleJDBCClient.java:409)
        at com.oceanbase.oms.connector.cache.AbstractTableSchemaCache.loadTableInfoFromDb(AbstractTableSchemaCache.java:189)
        at com.oceanbase.oms.connector.cache.AbstractTableSchemaCache.getSchema(AbstractTableSchemaCache.java:129)
        at com.oceanbase.oms.connector.cache.TableSchemaCacheManager.getSchema(TableSchemaCacheManager.java:27)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.tryBatchRecord(Writer.java:461)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.batchFlushDML(Writer.java:253)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushFullInsertBatch(Writer.java:230)
        at com.oceanbase.oms.connector.jdbc.sink.Writer.flushRecords(Writer.java:195)
        at com.oceanbase.oms.connector.jdbc.sink.DefaultJDBCSink.offer(DefaultJDBCSink.java:60)
        at com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSink.offer(OBOracleJDBCSink.java:25)
        at com.oceanbase.connector.framework.threadmanager.sinktask.SyncSinkConnectorTask.run(SyncSinkConnectorTask.java:44)
        at java.lang.Thread.run(Thread.java:853)
``` 

回放时, 获取某行数据对应的schema的信息

TODO: 

  - 获取各环节的堆积状态

# TODO

  - 断点续传机制

  - 一致性机制

# 机制分析

## sourceTask

任务启动器: DataFlowSourceBooster

  - 获取 表的元信息 (com.oceanbase.oms.connector.source.dataflow.DataFlowSourceBooster#initTables)
  - 启动 数据切分器 (com.oceanbase.oms.connector.source.dataflow.DataFlowSliceProvider)
    - 在断点续传的场景下, 需要注意schema的一致性?? (com.oceanbase.oms.connector.source.dataflow.SliceTaskManager#filterNeedPullTableInfos)
    - 启动slice-worker 线程池, 完成切分任务

全量复制的定制构建器: FrameworkCustomize, 一共有两个作用: 

  1. 管理 全量复制的一些特有机制 (FullFrameworkCustomize#coordinate)
     1. 配置预处理(JDBCPreprocessor), 根据目标数据库的配置, 变更 复制通路的配置: (JDBCPreprocessor#setConfigByDBConfig)
        1. 比如: 根据max_allowed_packet 调整 配置 sourceBatchMemorySize
     2. 心跳机制: 向文件系统维护 心跳文件 (HeartBeatManager)
        1. 样例: 

```
[root@R740-26 1]# pwd
/u01/ds/run/10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002/migrate/1
[root@R740-26 1]# grep '' heartbeat
{"accumulatedRunningTime":397,"capacity":5089999,"consistentQuantity":0,"dstRps":10313,"dstRpsRef":8000,"dstRt":0,"dstRtRef":1,"dstSpeed":7643410,"failedTables":0,"finishedTables":9,"firstRoundStartingGmtTime":1701160659,"id":"1","inconsistentQuantity":0,"message":"","numberOfDeletedRecords":0,"numberOfInsertedRecords":5089999,"numberOfUpdatedRecords":0,"predictedTimeToFinish":0,"processedRecords":5089999,"progress":"1.0","recordProgress":"1.0","reportingGmtTime":1701161056,"rps":10313,"srcRps":10313,"srcRpsRef":8000,"srcRt":0,"srcRtRef":1,"srcSpeed":7643410,"srcSpeedRef":8388608,"startingGmtTime":1701160659,"status":"done","subId":"1","totalTables":9,"type":"migrate"}
``` 
     3. 任务状态文件: 向文件系统维护 任务的状态文件 (OverviewManager)
        1. 样例: 

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
     4. 断点续传机制
        1. checkpoint文件: 

```
[root@R740-26 10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002]# pwd
/u01/ds/run/10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002
[root@R740-26 10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002]# grep '' oms.connector.checkpoint
{"checkpoint":"{}","version":"3.4.0","port":16001,"gmt":1701161062,"status":"done","firstRoundStartingGmtTime":1701160659,"startingGmtTime":1701160665,"finishedTables":9,"totalTables":9,"failedTables":0,"accumulatedRunningTime":397,"predictedTimeToFinish":0,"progress":"1.0","recordProgress":"1.0","processedRecords":5089999,"capacity":5089999,"inconsistentQuantity":0,"consistentQuantity":0,"numberOfInsertedRecords":5089999,"numberOfDeletedRecords":0,"numberOfUpdatedRecords":0}
[root@R740-26 10.186.16.126-9000:connector_v2:np_59s1c3qy1kkg-full_trans-1-0:0008000002]#
``` 
     5. 表缓存 (JDBCTableCache)
        1. 作用: ?
  2. 生成 数据处理器 (ETLBatchProcessor) (FullFrameworkCustomize#frameworkComponents)
     1. 按需构建 ETL各阶段:
        1. DDL转换阶段
        2. DML转换阶段
        3. 全局过滤阶段 (过滤数据变更类型: INSERT/UPDATE/DELET, 与全量无关)
     2. 链路的最后一级是 recordDispatcher, 与增量链路的处理器13 作用不同
        1. 全量任务默认使用: DynamicShardDispatcher (初始化位置: com.oceanbase.oms.connector.fullcustomize.FullFrameworkCustomize#buildRecordDispatcher)
           1. 与增量任务不同, 该处理器 不进行 事务重排; 而是计算分片表的路由, 将记录分发到分片表中

任务线程: ConditionedSourceConnectorTask

  - 创建线程的代码: com.oceanbase.connector.framework.threadmanager.sourcetask.SourceTaskManager#createConnectorTask
  - 协调 数据切分器/数据源/数据处理器/输出队列:
    - 从数据切分器中获取切分条件
    - 根据切分条件, 从数据源中获取数据
    - 使用数据处理器处理数据
    - 将处理后的数据 输出到 输出队列

数据切分器:

  - 切分方式: All / PK / PKIncrement / Block
  - 此处 对于BlockSliceService的逻辑解释 (com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient#blockSlice): 

  1.      1. 设置 优化器版本: alter session set OPTIMIZER_FEATURES_ENABLE= ?
     2. 判断表 是否是 bigfile tablespace: SELECT RELATIVE_FNO FROM DBA_EXTENTS WHERE OWNER='{}' AND SEGMENT_NAME='{}' AND ROWNUM<2, 
        1. 判断RELATIVE_FNO是否为1024 (不明确其中的逻辑关系), 如果是1024, 则为bigfile table
        2. 对于起始的块号, 需要转换成bigfile的块号 (com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient#smallFileFnoBlockNumberToBigfile )
     3. 查询块号: 
        1. SQL (如果是分区表, 还会增加partition的相关条件) (com.oceanbase.oms.dataflow.jdbcclient.oraclebased.oracle.OracleJDBCClient#createBlockQuerySql)

```
SELECT  OMS_OBJECT_NUMBER, OMS_RELATIVE_FNO, OMS_BLOCK_NUMBER, OMS_ROW_NUMBER, BLOCKS FROM ( SELECT OBJ.DATA_OBJECT_ID OMS_OBJECT_NUMBER,EXT.RELATIVE_FNO OMS_RELATIVE_FNO, EXT.BLOCK_ID OMS_BLOCK_NUMBER, 0 OMS_ROW_NUMBER , EXT.BLOCKS BLOCKS   FROM DBA_EXTENTS EXT, ALL_OBJECTS OBJ WHERE EXT.OWNER = 'OBTEST' AND EXT.OWNER = OBJ.OWNER AND EXT.SEGMENT_NAME = 'ORDER_LINE'
AND EXT.SEGMENT_TYPE IN ('TABLE', 'TABLE PARTITION','TABLE SUBPARTITION') AND EXT.SEGMENT_NAME = OBJ.OBJECT_NAME   ) ORDER BY OMS_OBJECT_NUMBER, OMS_RELATIVE_FNO, OMS_BLOCK_NUMBER
 
 
---
输出样例: 
 
OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	      19552		 0	    8
	    76327		 5	      67984		 0	    8
	    76327		 5	      67992		 0	    8
	    76327		 5	      68008		 0	    8
	    76327		 5	      68016		 0	    8
	    76327		 5	      68024		 0	    8
	    76327		 5	      68040		 0	    8
	    76327		 5	      68048		 0	    8
	    76327		 5	      68056		 0	    8
	    76327		 5	      68072		 0	    8
	    76327		 5	      68080		 0	    8

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	      68088		 0	    8
	    76327		 5	      88456		 0	    8
	    76327		 5	      88472		 0	    8
	    76327		 5	      88488		 0	    8
	    76327		 5	      88504		 0	    8
	    76327		 5	      88704		 0	  128
	    76327		 5	      88832		 0	  128
	    76327		 5	      89600		 0	  128
	    76327		 5	      89728		 0	  128
	    76327		 5	      90240		 0	  128
	    76327		 5	      92544		 0	  128

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	      92800		 0	  128
	    76327		 5	      93440		 0	  128
	    76327		 5	      93568		 0	  128
	    76327		 5	      93696		 0	  128
	    76327		 5	      93824		 0	  128
	    76327		 5	      93952		 0	  128
	    76327		 5	      94592		 0	  128
	    76327		 5	      94720		 0	  128
	    76327		 5	      95104		 0	  128
	    76327		 5	      95232		 0	  128
	    76327		 5	      95360		 0	  128

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	      97920		 0	  128
	    76327		 5	      98176		 0	  128
	    76327		 5	      98688		 0	  128
	    76327		 5	      98816		 0	  128
	    76327		 5	      99072		 0	  128
	    76327		 5	      99200		 0	  128
	    76327		 5	      99456		 0	  128
	    76327		 5	      99712		 0	  128
	    76327		 5	      99840		 0	  128
	    76327		 5	      99968		 0	  128
	    76327		 5	     100352		 0	  128

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	     100480		 0	  128
	    76327		 5	     100864		 0	  128
	    76327		 5	     100992		 0	  128
	    76327		 5	     101120		 0	  128
	    76327		 5	     101248		 0	  128
	    76327		 5	     101632		 0	  128
	    76327		 5	     102272		 0	  128
	    76327		 5	     102400		 0	  128
	    76327		 5	     102528		 0	  128
	    76327		 5	     103424		 0	  128
	    76327		 5	     103552		 0	  128

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	     103808		 0	  128
	    76327		 5	     103936		 0	  128
	    76327		 5	     104064		 0	  128
	    76327		 5	     104192		 0	  128
	    76327		 5	     104320		 0	  128
	    76327		 5	     104448		 0	  128
	    76327		 5	     104576		 0	  128
	    76327		 5	     104704		 0	  128
	    76327		 5	     105344		 0	  128
	    76327		 5	     105472		 0	  128
	    76327		 5	     105600		 0	  128

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	     105856		 0	  128
	    76327		 5	     105984		 0	  128
	    76327		 5	     106240		 0	  128
	    76327		 5	     107136		 0	  128
	    76327		 5	     107264		 0	  128
	    76327		 5	     107392		 0	  128
	    76327		 5	     107520		 0	  128
	    76327		 5	     107648		 0	  128
	    76327		 5	     107776		 0	  128
	    76327		 5	     107904		 0	  128
	    76327		 5	     108032		 0	  128

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	     108416		 0	  128
	    76327		 5	     108544		 0	  128
	    76327		 5	     108800		 0	 1024
	    76327		 5	     109824		 0	 1024
	    76327		 5	     110848		 0	 1024
	    76327		 5	     114560		 0	 1024
	    76327		 5	     116224		 0	 1024
	    76327		 5	     118016		 0	 1024
	    76327		 5	     125440		 0	 1024
	    76327		 5	     177280		 0	 1024
	    76327		 5	     178560		 0	 1024

OMS_OBJECT_NUMBER OMS_RELATIVE_FNO OMS_BLOCK_NUMBER OMS_ROW_NUMBER     BLOCKS
----------------- ---------------- ---------------- -------------- ----------
	    76327		 5	     180736		 0	 1024
	    76327		 5	     182784		 0	 1024
	    76327		 5	     183808		 0	 1024
	    76327		 5	     184832		 0	 1024
	    76327		 5	     185984		 0	 1024
	    76327		 5	     187008		 0	 1024
	    76327		 5	     188032		 0	 1024
	    76327		 5	     190080		 0	 1024
	    76327		 5	     193408		 0	 1024
``` 
     4. 切分: OMS_BLOCK_NUMBER 为起始地址, BLOCKS为数量, 切分大小为 配置limitator.splitor.blocks

数据源: DataFlowSource (由DataFlowSourceBooster指定)

  - 数据源需要从数据库中拉取数据: com.oceanbase.oms.connector.source.dataflow.DataFlowSource#poll(com.oceanbase.oms.connector.common.querycondition.QueryCondition
  - 拉取的条件是 从数据切分器中获取 (com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask#run)
  - 数据拉取逻辑: 获取一个 切片 的数据 (com.oceanbase.oms.connector.source.dataflow.DataFlowSource#poll)
    - 使用 fetcher, 下SQL
      - 对于有一致性的表, 使用 jdbcClient.queryDataBySliceConsistency
        - 仅对MySQL有效, 设置隔离级别为RR, 对表上读锁, 然后下SQL (com.oceanbase.oms.dataflow.jdbcclient.mysqlbased.mysql.MysqlQueryClient#queryDataBySliceConsistency)
      - 否则, 使用 jdbcClient.queryDataBySlice
      - SQL格式: 

```
"SELECT /*+ ROWID(\"%s\") */ %s FROM %s %s WHERE ROWID>? AND ROWID<=?"
``` 
    - 使用 转换函数, 将SQL结果转成对象 (com.oceanbase.oms.connector.source.dataflow.RecordValueConvertFunction)
    - 构建一个 StreamRecordBatchBuilder, 将 fetcher和转换函数 组织在一起, 包装成一个 数据行的迭代器

数据处理器: 

  - 处理函数: ConditionedSourceConnectorTask#handleRecordBatch
  - 处理器类型: com.oceanbase.oms.connector.customize.batchprocessor.ETLBatchProcessor (与增量任务中的ETL过程一致)

目标队列: 

  - 处理函数: ConditionedSourceConnectorTask#handleRecordBatch
  - 队列类型: com.oceanbase.connector.framework.threadmanager.QueuedSlot

## sinkTask

任务启动器: com.oceanbase.oms.connector.jdbc.sink.oboracle.OBOracleJDBCSinkBooster

任务结构与增量任务处理器14一致

### 水位监视器

启动: 由OBOracleJDBCSinkBooster#startOBHighWatermarkMonitor启动

监控目标数据库的 CPU和内存使用的水位 (通过SQL), 如果水位超过阈值, 记录日志

## 限流机制 (ThrottleCenter)

限流入口

  - 堆栈

```
ts=2023-12-03 11:59:56;thread_name=sourceTask-7;id=43;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@7852e922
    @com.oceanbase.oms.connector.common.ThrottleCenter.inThrottle()
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.handleRecordBatch(ConditionedSourceConnectorTask.java:183)
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.pollQueryConditionWithRetry(ConditionedSourceConnectorTask.java:125)
        at com.oceanbase.connector.framework.threadmanager.sourcetask.ConditionedSourceConnectorTask.run(ConditionedSourceConnectorTask.java:68)
        at java.lang.Thread.run(Thread.java:853)
``` 

限流出口

  - 堆栈

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

# 结论

本文记述分析步骤和细节. 梳理后整理到:

[20231204 - 整理OMS的全量复制链路]
