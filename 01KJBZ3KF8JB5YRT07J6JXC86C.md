---
title: 20231031 - 诊断OMS界面流程
confluence_page_id: 2589337
created_at: 2023-10-31T05:53:21+00:00
updated_at: 2023-10-31T05:53:21+00:00
---

现象: 界面上启动store, 操作提示成功, 但实际没有生效

在oms-web.log中, 依靠操作编号3000001找到记录: 

```
dss-operation.log:2023-10-31 13:33:56.126 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request start. id 3000001
dss-operation.log:2023-10-31 13:33:57.235 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request [3000001] started
dss-operation.log:2023-10-31 13:33:57.237 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start to execute node, id: 3000001, title: STORE_RESTART_TASK，requestId: 3000001
dss-operation.log:2023-10-31 13:33:58.411 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] nodeExecute started. nodeId 3000001
dss-operation.log:2023-10-31 13:33:58.720 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start to execute order, id: 3000001, title: STORE_RESTART_TASK
dss-operation.log:2023-10-31 13:33:58.861 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] orderExecute started. orderId 3000001, requestId: 3000001
dss-operation.log:2023-10-31 13:34:04.800 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start to execute step, id 3000001, title: CLEAR_CRAWLER
dss-operation.log:2023-10-31 13:34:04.800 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] stepType CLEAR_CRAWLER start. orderId 3000001, stepId 3000001
dss-operation.log:2023-10-31 13:34:51.909 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] step processed. id: 3000001, cost: 53031 ms
dss-operation.log:2023-10-31 13:34:52.353 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] step executes successfully, id: 3000001, title: CLEAR_CRAWLER
dss-operation.log:2023-10-31 13:34:52.749 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] stepType RESTORE_CRAWLER start. orderId 3000001, stepId 3000002
dss-operation.log:2023-10-31 13:35:34.489 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] orderCallback with orderId 3000001, res true
dss-operation.log:2023-10-31 13:35:35.870 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] order executes successfully, id: 3000001, title: STORE_RESTART_TASK
dss-operation.log:2023-10-31 13:35:35.870 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] nodecallback nodeId [3000001], res: [true]
dss-operation.log:2023-10-31 13:35:36.995 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] node id 3000001 executes successfully, title: STORE_RESTART_TASK
dss-operation.log:2023-10-31 13:35:41.997 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request [3000001] started
dss-operation.log:2023-10-31 13:35:42.000 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] all nodes of request 3000001 success
dss-operation.log:2023-10-31 13:35:42.654 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request id: 3000001 success
grep: event-tracking: Is a directory
oms-web.log:2023-10-31 13:33:58.482 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 159 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] RESPONSE : OmsApiReturnResult(success=true, errorDetail=null, code=null, message=null, advice=null, requestId=f4662b9d-2c09-4810-9e6c-af81bdcc3391, pageNumber=null, pageSize=null, totalCount=null, cost=null, data=3000001)
oms-web.log:2023-10-31 13:36:54.105 [http-nio-8090-exec-9] INFO  c.a.o.c.a.WebLogAspect 159 - [65c27cc0-f6a4-47e7-a521-5d34f4f2e7bb] RESPONSE : OmsApiReturnResult(success=true, errorDetail=null, code=null, message=null, advice=null, requestId=65c27cc0-f6a4-47e7-a521-5d34f4f2e7bb, pageNumber=1, pageSize=10, totalCount=9, cost=null, data=[RequestDescribeResponse(id=3000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698730430000, finishedTime=1698730542000, nodes=null), RequestDescribeResponse(id=2000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698728689000, finishedTime=1698728799000, nodes=null), RequestDescribeResponse(id=2000002, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727460000, finishedTime=1698727573000, nodes=null), RequestDescribeResponse(id=2000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727199000, finishedTime=1698727317000, nodes=null), RequestDescribeResponse(id=1000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698654582000, finishedTime=1698654669000, nodes=null), RequestDescribeResponse(id=1000002, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=PROCESSING, createTime=1698653982000, finishedTime=null, nodes=null), RequestDescribeResponse(id=1000001, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698653936000, finishedTime=1698653970000, nodes=null), RequestDescribeResponse(id=2, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229562000, finishedTime=1698229572000, nodes=null), RequestDescribeResponse(id=1, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229521000, finishedTime=1698229536000, nodes=null)])
oms-web.log:2023-10-31 13:37:06.096 [http-nio-8090-exec-7] INFO  c.a.o.c.a.WebLogAspect 159 - [d64ffed0-9ff3-4b17-b08e-229e6060eebf] RESPONSE : OmsApiReturnResult(success=true, errorDetail=null, code=null, message=null, advice=null, requestId=d64ffed0-9ff3-4b17-b08e-229e6060eebf, pageNumber=1, pageSize=10, totalCount=9, cost=null, data=[RequestDescribeResponse(id=3000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698730430000, finishedTime=1698730542000, nodes=null), RequestDescribeResponse(id=2000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698728689000, finishedTime=1698728799000, nodes=null), RequestDescribeResponse(id=2000002, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727460000, finishedTime=1698727573000, nodes=null), RequestDescribeResponse(id=2000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727199000, finishedTime=1698727317000, nodes=null), RequestDescribeResponse(id=1000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698654582000, finishedTime=1698654669000, nodes=null), RequestDescribeResponse(id=1000002, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=PROCESSING, createTime=1698653982000, finishedTime=null, nodes=null), RequestDescribeResponse(id=1000001, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698653936000, finishedTime=1698653970000, nodes=null), RequestDescribeResponse(id=2, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229562000, finishedTime=1698229572000, nodes=null), RequestDescribeResponse(id=1, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229521000, finishedTime=1698229536000, nodes=null)])
oms-web.log:2023-10-31 13:37:20.301 [http-nio-8090-exec-10] INFO  c.a.o.c.a.WebLogAspect 159 - [8f1cb543-813f-4850-8abf-a1199605b8d4] RESPONSE : OmsApiReturnResult(success=true, errorDetail=null, code=null, message=null, advice=null, requestId=8f1cb543-813f-4850-8abf-a1199605b8d4, pageNumber=1, pageSize=10, totalCount=9, cost=null, data=[RequestDescribeResponse(id=3000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698730430000, finishedTime=1698730542000, nodes=null), RequestDescribeResponse(id=2000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698728689000, finishedTime=1698728799000, nodes=null), RequestDescribeResponse(id=2000002, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727460000, finishedTime=1698727573000, nodes=null), RequestDescribeResponse(id=2000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727199000, finishedTime=1698727317000, nodes=null), RequestDescribeResponse(id=1000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698654582000, finishedTime=1698654669000, nodes=null), RequestDescribeResponse(id=1000002, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=PROCESSING, createTime=1698653982000, finishedTime=null, nodes=null), RequestDescribeResponse(id=1000001, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698653936000, finishedTime=1698653970000, nodes=null), RequestDescribeResponse(id=2, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229562000, finishedTime=1698229572000, nodes=null), RequestDescribeResponse(id=1, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229521000, finishedTime=1698229536000, nodes=null)])
oms-web.log:2023-10-31 13:38:40.938 [http-nio-8090-exec-8] INFO  c.a.o.c.a.WebLogAspect 159 - [a8bb2907-4a70-438d-a6ee-99570409fc84] RESPONSE : OmsApiReturnResult(success=true, errorDetail=null, code=null, message=null, advice=null, requestId=a8bb2907-4a70-438d-a6ee-99570409fc84, pageNumber=1, pageSize=10, totalCount=9, cost=null, data=[RequestDescribeResponse(id=3000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698730430000, finishedTime=1698730542000, nodes=null), RequestDescribeResponse(id=2000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698728689000, finishedTime=1698728799000, nodes=null), RequestDescribeResponse(id=2000002, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727460000, finishedTime=1698727573000, nodes=null), RequestDescribeResponse(id=2000001, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=ERROR, createTime=1698727199000, finishedTime=1698727317000, nodes=null), RequestDescribeResponse(id=1000003, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, type=重启 store, operator=admin, target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, status=SUCCESS, createTime=1698654582000, finishedTime=1698654669000, nodes=null), RequestDescribeResponse(id=1000002, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=PROCESSING, createTime=1698653982000, finishedTime=null, nodes=null), RequestDescribeResponse(id=1000001, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698653936000, finishedTime=1698653970000, nodes=null), RequestDescribeResponse(id=2, title=OPERATOR_SUBASSEMBLY_START ConnectorTask-NAME, type=启动 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229562000, finishedTime=1698229572000, nodes=null), RequestDescribeResponse(id=1, title=OPERATOR_SUBASSEMBLY_STOP ConnectorTask-NAME, type=停止 Incr-Sync, operator=admin, target=10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002, status=SUCCESS, createTime=1698229521000, finishedTime=1698229536000, nodes=null)])
grep: upgrade_4.2.0: Is a directory
``` 

确定trace ID, 并在/home/admin/logs/ghana/Ghana下进行grep: 

```
[root@R740-26 Ghana]# grep af81bdcc3391 *
common-default.log:2023-10-31 13:33:50.597 [http-nio-8090-exec-3] INFO  c.a.o.c.s.f.d.StoreDescribeDTO 285 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] Store status sample: crawler_name 10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, crawler_status=1, status=1, active=true => output_status=UNEXPECTED_EXIT
dss-operation.log:2023-10-31 13:33:50.626 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start generate ordxer. commonParam CommonParam(cmUrl=http://10.186.16.126:8088, clusterName=null, instanceId=null, creator=admin, title=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESSNAME, msg=OPERATOR_SUBASSEMBLY_START store OPERATOR_SUBASSEMBLY_PULL_PROCESS, param=StoreCommonParam(crawler=null, async=false, clearCrawler=true, useOriginPort=false, allowStoreNotExist=null, crawlerNames=[10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001]), target=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, affectedProjectIds=[np_56el1g1jvrs0]), jobType STORE_RESTART_TASK
dss-operation.log:2023-10-31 13:33:56.126 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request start. id 3000001
dss-operation.log:2023-10-31 13:33:57.235 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request [3000001] started
dss-operation.log:2023-10-31 13:33:57.237 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start to execute node, id: 3000001, title: STORE_RESTART_TASK，requestId: 3000001
dss-operation.log:2023-10-31 13:33:58.411 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] nodeExecute started. nodeId 3000001
dss-operation.log:2023-10-31 13:33:58.720 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start to execute order, id: 3000001, title: STORE_RESTART_TASK
dss-operation.log:2023-10-31 13:33:58.861 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] orderExecute started. orderId 3000001, requestId: 3000001
dss-operation.log:2023-10-31 13:34:04.800 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start to execute step, id 3000001, title: CLEAR_CRAWLER
dss-operation.log:2023-10-31 13:34:04.800 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] stepType CLEAR_CRAWLER start. orderId 3000001, stepId 3000001
dss-operation.log:2023-10-31 13:34:51.909 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] step processed. id: 3000001, cost: 53031 ms
dss-operation.log:2023-10-31 13:34:52.353 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] step executes successfully, id: 3000001, title: CLEAR_CRAWLER
dss-operation.log:2023-10-31 13:34:52.749 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] start to execute step, id 3000002, title: RESTORE_CRAWLER
dss-operation.log:2023-10-31 13:34:52.749 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] stepType RESTORE_CRAWLER start. orderId 3000001, stepId 3000002
dss-operation.log:2023-10-31 13:34:52.791 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] useOriginPort: false
dss-operation.log:2023-10-31 13:35:32.625 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] step processed. id: 3000002, cost: 40272 ms
dss-operation.log:2023-10-31 13:35:34.489 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] step executes successfully, id: 3000002, title: RESTORE_CRAWLER
dss-operation.log:2023-10-31 13:35:34.489 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] orderCallback with orderId 3000001, res true
dss-operation.log:2023-10-31 13:35:35.870 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] order executes successfully, id: 3000001, title: STORE_RESTART_TASK
dss-operation.log:2023-10-31 13:35:35.870 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] nodecallback nodeId [3000001], res: [true]
dss-operation.log:2023-10-31 13:35:36.995 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] node id 3000001 executes successfully, title: STORE_RESTART_TASK
dss-operation.log:2023-10-31 13:35:41.997 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request [3000001] started
dss-operation.log:2023-10-31 13:35:42.000 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] all nodes of request 3000001 success
dss-operation.log:2023-10-31 13:35:42.654 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] request id: 3000001 success
grep: event-tracking: Is a directory
meta-db.log:2023-10-31 13:33:50.713 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [RequestV3DAO.saveRequestV2] [N] [75]
meta-db.log:2023-10-31 13:33:50.799 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [RequestProjectRelationDAO.save] [N] [86]
meta-db.log:2023-10-31 13:33:51.133 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [NodeV2Dao.saveNode] [N] [333]
meta-db.log:2023-10-31 13:33:52.488 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [OrderV2Dao.saveOrder] [N] [1354]
meta-db.log:2023-10-31 13:33:53.644 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [StepV2Dao.saveStepV2] [N] [1149]
meta-db.log:2023-10-31 13:33:53.694 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [RequestV3DAO.updateRequestV2] [N] [40]
meta-db.log:2023-10-31 13:33:56.693 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [LockDao.saveLock] [N] [562]
meta-db.log:2023-10-31 13:33:57.161 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [RequestV3DAO.updateRequestV2] [N] [152]
meta-db.log:2023-10-31 13:33:58.411 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [NodeV2Dao.updateNodeV2] [N] [1174]
meta-db.log:2023-10-31 13:33:58.720 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [OrderV2Dao.updateOrderV2] [N] [306]
meta-db.log:2023-10-31 13:34:04.800 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [StepV2Dao.updateStepV2] [N] [5889]
meta-db.log:2023-10-31 13:34:52.353 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [StepV2Dao.updateStepV2] [N] [444]
meta-db.log:2023-10-31 13:34:52.748 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [StepV2Dao.updateStepV2] [N] [395]
meta-db.log:2023-10-31 13:34:52.791 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [OrderV2Dao.findByOrderId] [N] [42]
meta-db.log:2023-10-31 13:35:34.489 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [StepV2Dao.updateStepV2] [N] [1864]
meta-db.log:2023-10-31 13:35:35.870 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [OrderV2Dao.updateOrderV2] [N] [1378]
meta-db.log:2023-10-31 13:35:36.995 [INFO] [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [NodeV2Dao.updateNodeV2] [N] [1118]
oms-integration.log:2023-10-31 13:33:50.582 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [GET] [http://10.186.16.126:8088] [/crawler/detail] [200] [11] []
oms-integration.log:2023-10-31 13:33:50.597 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [GET] [http://10.186.16.126:8088] [/monitor/crawler/overview] [200] [11] []
oms-integration.log:2023-10-31 13:33:50.624 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [GET] [http://10.186.16.126:8088] [/monitor/crawler/overview] [200] [15] []
oms-integration.log:2023-10-31 13:34:26.908 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [POST] [http://10.186.16.126:8088] [/crawler/clear] [200] [22106] []
oms-integration.log:2023-10-31 13:35:32.625 [INFO] - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] [GET] [http://10.186.16.126:8088] [/crawler/restore] [200] [39834] []
oms-web.log:2023-10-31 13:33:50.568 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 103 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] URL = http://localhost:8090/omsp/operator/NAME/stores/10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001/start
oms-web.log:2023-10-31 13:33:50.569 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 104 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HTTP_METHOD = POST
oms-web.log:2023-10-31 13:33:50.569 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 105 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] IP = 192.168.22.8
oms-web.log:2023-10-31 13:33:50.570 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 110 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] METHOD = com.alipay.oms.priv.controller.OmsOperatorStoreController.startStore
oms-web.log:2023-10-31 13:33:50.570 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 113 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ARGS = ["NAME","10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001","default"]
oms-web.log:2023-10-31 13:33:50.570 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 124 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] Occur exception when parse json array to acquire uid, args: ["NAME","10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001","default"]. ignore it.
oms-web.log:2023-10-31 13:33:58.482 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 159 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] RESPONSE : OmsApiReturnResult(success=true, errorDetail=null, code=null, message=null, advice=null, requestId=f4662b9d-2c09-4810-9e6c-af81bdcc3391, pageNumber=null, pageSize=null, totalCount=null, cost=null, data=3000001)
oms-web.log:2023-10-31 13:33:58.483 [http-nio-8090-exec-3] INFO  c.a.o.c.a.WebLogAspect 160 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] EXECUTE METHOD com.alipay.oms.priv.controller.OmsOperatorStoreController.startStore SPEND TIME 7914 MS
grep: upgrade_4.2.0: Is a directory
[root@R740-26 Ghana]#
``` 

在 oms-integration中, 看到Ghana调用了接口: [<http://10.186.16.126:8088>] [/crawler/clear] 和 [/crawler/restore]

在CM日志中, /home/admin/logs/cm/log, 进行grep: 

```
cm-web.log:2023-10-31 13:34:14 INFO com.alibaba.drc.common.utils.HttpClientPoolUtils.grabAsHttpResponsePostCustom(HttpClientPoolUtils.java:604) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] post POST http://10.186.16.126:9000/operateService?action=clear HTTP/1.1 sourceUrl http://10.186.16.126:9000/operateService?action=clear
cm-web.log:2023-10-31 13:34:14 INFO com.alibaba.drc.common.utils.HttpClientPoolUtils.process(HttpClientPoolUtils.java:632) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] source Url in process http://10.186.16.126:9000/operateService?action=clear
cm-web.log:2023-10-31 13:34:14 INFO com.alibaba.drc.common.utils.HttpClientPoolUtils.process(HttpClientPoolUtils.java:653) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] http context org.apache.http.client.protocol.HttpClientContext@4a62f6 source url http://10.186.16.126:9000/operateService?action=clear
cm-web.log:2023-10-31 13:34:14 WARN com.alibaba.drc.supervisor.service.impl.SupervisorServiceImpl.sendJobToSupervisor(SupervisorServiceImpl.java:584) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] sendHttpPostRequest url http://10.186.16.126:9000/operateService?action=clear, result {"code":200,"message":"success","status":"ok","resultData":null,"errorDetail":null}
cm-web.log:2023-10-31 13:34:53 INFO com.alibaba.drc.supervisor.SupervisorCommandInvoker.cleanStoreErrorOnSupervior(SupervisorCommandInvoker.java:141) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] Success to clean error for store: 10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001
cm-web.log:2023-10-31 13:35:03 INFO com.alibaba.drc.common.utils.HttpClientPoolUtils.grabAsHttpResponsePostCustom(HttpClientPoolUtils.java:604) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] post POST http://10.186.16.126:9000/operateService?action=start HTTP/1.1 sourceUrl http://10.186.16.126:9000/operateService?action=start
cm-web.log:2023-10-31 13:35:03 INFO com.alibaba.drc.common.utils.HttpClientPoolUtils.process(HttpClientPoolUtils.java:632) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] source Url in process http://10.186.16.126:9000/operateService?action=start
cm-web.log:2023-10-31 13:35:03 INFO com.alibaba.drc.common.utils.HttpClientPoolUtils.process(HttpClientPoolUtils.java:653) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] http context org.apache.http.client.protocol.HttpClientContext@61d420f3 source url http://10.186.16.126:9000/operateService?action=start
cm-web.log:2023-10-31 13:35:03 WARN com.alibaba.drc.supervisor.service.impl.SupervisorServiceImpl.sendJobToSupervisor(SupervisorServiceImpl.java:584) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] sendHttpPostRequest url http://10.186.16.126:9000/operateService?action=start, result {"code":200,"message":"success","status":"ok","resultData":null,"errorDetail":null}
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.selectCrawlerByName,Y,2
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HaDao.selectNewestHeartBeatByTaskName,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HaDao.selectSCheckPointByCrawlerName,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskId,Y,4
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectSubTopicsByParam,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.selectCrawlerByNameList,Y,2
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ProcessOverviewDao.selectProcessOverviewByNameList,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HaDao.selectHeartBeatsByTaskNames,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HaDao.selectHeartBeatsByTaskNames,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectDistinctByType,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectSubTopicsByParam,Y,6
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.selectCrawlerByNameList,Y,2
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ProcessOverviewDao.selectProcessOverviewByNameList,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HaDao.selectHeartBeatsByTaskNames,Y,1
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HaDao.selectHeartBeatsByTaskNames,Y,0
dao-digest.log:2023-10-31 13:33:50 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectDistinctByType,Y,1
dao-digest.log:2023-10-31 13:34:04 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectSubTopicByName,Y,1
dao-digest.log:2023-10-31 13:34:04 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.selectCrawlerByName,Y,1
dao-digest.log:2023-10-31 13:34:11 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.createTaskPool,Y,7179
dao-digest.log:2023-10-31 13:34:12 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskStepByTemplateName,Y,2
dao-digest.log:2023-10-31 13:34:12 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:34:12 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.selectCrawlerById,Y,2
dao-digest.log:2023-10-31 13:34:12 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectSubTopicByName,Y,1
dao-digest.log:2023-10-31 13:34:12 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskId,Y,1
dao-digest.log:2023-10-31 13:34:12 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskId,Y,1
dao-digest.log:2023-10-31 13:34:14 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.createTaskStep,Y,2092
dao-digest.log:2023-10-31 13:34:17 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.updateTaskStep,Y,3389
dao-digest.log:2023-10-31 13:34:18 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.updateTaskPool,Y,1233
dao-digest.log:2023-10-31 13:34:18 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,2
dao-digest.log:2023-10-31 13:34:20 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,2
dao-digest.log:2023-10-31 13:34:22 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:34:24 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:34:26 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,2
dao-digest.log:2023-10-31 13:34:26 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.updateCrawler,Y,45
dao-digest.log:2023-10-31 13:34:52 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectSubTopicByName,Y,1
dao-digest.log:2023-10-31 13:34:52 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.selectCrawlerByName,Y,166
dao-digest.log:2023-10-31 13:34:52 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.findActiveCrawlersBySubtopic,Y,1
dao-digest.log:2023-10-31 13:34:53 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.updateCrawlerErrorDetails,Y,946
dao-digest.log:2023-10-31 13:34:53 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] HaDao.selectSCheckPointByCrawlerName,Y,1
dao-digest.log:2023-10-31 13:34:53 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskIdLimit1,Y,1
dao-digest.log:2023-10-31 13:34:56 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.updateConfigTaskValueIdByTaskId,Y,2685
dao-digest.log:2023-10-31 13:34:56 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskIdLimit1,Y,2
dao-digest.log:2023-10-31 13:35:00 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.updateConfigTaskValueIdByTaskId,Y,4165
dao-digest.log:2023-10-31 13:35:00 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskIdLimit1,Y,4
dao-digest.log:2023-10-31 13:35:01 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.updateConfigTaskValueIdByTaskId,Y,787
dao-digest.log:2023-10-31 13:35:01 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskIdLimit1,Y,1
dao-digest.log:2023-10-31 13:35:01 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.updateConfigTaskValueIdByTaskId,Y,323
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.updateCrawler,Y,232
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.createTaskPool,Y,644
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskStepByTemplateName,Y,1
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,2
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] CrawlerDao.selectCrawlerById,Y,1
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TopicDao.selectSubTopicByName,Y,1
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskId,Y,1
dao-digest.log:2023-10-31 13:35:02 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] ConfigDao.selectConfigTaskByTaskId,Y,2
dao-digest.log:2023-10-31 13:35:03 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.createTaskStep,Y,923
dao-digest.log:2023-10-31 13:35:04 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.updateTaskStep,Y,824
dao-digest.log:2023-10-31 13:35:05 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.updateTaskPool,Y,858
dao-digest.log:2023-10-31 13:35:05 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:07 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,2
dao-digest.log:2023-10-31 13:35:09 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:11 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:13 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:15 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,2
dao-digest.log:2023-10-31 13:35:17 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:19 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:21 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,2
dao-digest.log:2023-10-31 13:35:23 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:25 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,0
dao-digest.log:2023-10-31 13:35:27 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1
dao-digest.log:2023-10-31 13:35:29 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,3
dao-digest.log:2023-10-31 13:35:32 - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] TaskDao.selectTaskPoolById,Y,1070
service.log:2023-10-31 13:33:50 INFO com.alibaba.drc.service.monitor.CrawlerMonitorService.getCrawlerOverview(CrawlerMonitorService.java:66) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] orderByParam crawlerDelay, orderByType desc
service.log:2023-10-31 13:33:50 INFO com.alibaba.drc.service.monitor.CrawlerMonitorService.getCrawlerOverview(CrawlerMonitorService.java:66) - [f4662b9d-2c09-4810-9e6c-af81bdcc3391] orderByParam crawlerDelay, orderByType desc
[root@R740-26 log]#
``` 

查看cm-web.log中, 与supervisor通讯, 调用了url: <http://10.186.16.126:9000/operateService?action=clear>, 和 <http://10.186.16.126:9000/operateService?action=start>

在supervisor日志中筛选: 

```
[root@R740-26 supervisor]# grep af81bdcc3391 *
command.log:[2023-10-31 13:34:53.917][INFO][http-nio-9000-exec-9][CommandInvoker:189][f4662b9d-2c09-4810-9e6c-af81bdcc3391][169873049300000|DELETE_FILE] Start to Invoke Command, response: DeleteFileCommand(path=/home/ds/store/store7100, name=error.details)
command.log:[2023-10-31 13:34:53.940][INFO][http-nio-9000-exec-9][DeleteFileCmdImpl:26][f4662b9d-2c09-4810-9e6c-af81bdcc3391][169873049300000|DELETE_FILE] Invoke DeleteFileCmd, actionId:169873049300000, DeleteFileCommand(path=/home/ds/store/store7100, name=error.details)
command.log:[2023-10-31 13:34:53.953][INFO][http-nio-9000-exec-9][CommandInvoker:189][f4662b9d-2c09-4810-9e6c-af81bdcc3391][169873049300001|CLEAN_ERROR] Start to Invoke Command, response: CleanErrorCommand(componentTaskName=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, moduleType=STORE, holdErrorUpdateTimeMs=1698730493907)
command.log:[2023-10-31 13:34:53.954][INFO][http-nio-9000-exec-9][CleanErrorCmdImpl:24][f4662b9d-2c09-4810-9e6c-af81bdcc3391][169873049300001|CLEAN_ERROR] Invoke CleanErrorCmd, actionId:169873049300001, CleanErrorCommand(componentTaskName=10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001, moduleType=STORE, holdErrorUpdateTimeMs=1698730493907)
monitor.log:[2023-10-31 13:34:53.952][f4662b9d-2c09-4810-9e6c-af81bdcc3391] [DELETE_FILE, 169873049300000] monitor value 35
monitor.log:[2023-10-31 13:34:53.956][f4662b9d-2c09-4810-9e6c-af81bdcc3391] [CLEAN_ERROR, 169873049300001] monitor value 3
routine.log:[2023-10-31 13:34:53.955][INFO][http-nio-9000-exec-9][CollectErrorRoutine:265][f4662b9d-2c09-4810-9e6c-af81bdcc3391] Updated reporting time for component error details: {INCR_TRANS={10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-incr_trans-1-0:0000000002=1698676557}, FULL_TRANS={10.186.16.126-9000:connector_v2:np_56el1g1jvrs0-full_trans-1-0:0000000001=-1}, STORE={10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001=1698730493907, 10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001.tmp=-1, 10.186.16.126-7101:ORACLE_np_56wquq78g37k_56wr24sdtun4-1-0:0001000001=-1}}
``` 

仅有clear操作的日志, 没有调用start的日志??
