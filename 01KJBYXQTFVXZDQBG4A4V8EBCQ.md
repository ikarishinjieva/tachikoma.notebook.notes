---
title: 20221105 - oceanbase集群恢复
confluence_page_id: 2130035
created_at: 2022-11-05T15:11:51+00:00
updated_at: 2022-11-08T11:10:11+00:00
---

从雷霞处接受损坏的ob集群, 进行恢复

# 诊断-1

从OCP启动停止的observer, 没有日志, 没有进程

手工执行 

```
cd /home/admin/oceanbase && /home/admin/oceanbase/bin/observer
``` 

可以启动observer, kill掉新启动的进程后, 界面显示: 

![image2022-11-5 23:24:15.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-5%2023%3A24%3A15.png)

需要搞明白这部分界面的逻辑

刷新架构图, 可以看到XHR的请求:

![image2022-11-8 10:51:57.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2010%3A51%3A57.png)

### 追踪query

在OCP日志中找到请求, 可以获得 traceID:

![image2022-11-5 23:41:37.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-5%2023%3A41%3A37.png)

追踪traceID, 可获得日志: 

```
2022-11-05 23:37:45.207  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.server.trace.RequestTracingAspect  : API: [GET /api/v2/ob/clusters/4000001?id=4000001, client=172.22.169.13, traceId=e1015c3362ef45a7, method=SuccessResponse com.alipay.ocp.server.controller.ObClusterController.getObCluster(Long), args=4000001,]
2022-11-05 23:37:45.861  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:45.953  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.ob.cluster.ClusterSyncService  : cluster is in operation, skip sync cluster, clusterId=4000001, status=OPERATING
2022-11-05 23:37:46.166  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.a.service.OcpAlarmSearchService  : alarms load from database, currentCount=27, historyCount=1
2022-11-05 23:37:46.321  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.323  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.323  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SELECT `zone`,                                   MAX(CASE `name` WHEN 'region' THEN `info` ELSE '' END ) `region`,                             MAX(CASE `name` WHEN 'idc' THEN `info` ELSE '' END ) `idc`,                                   MAX(CASE `name` WHEN 'status' THEN `info` ELSE '' END ) `status`,                             MAX(CASE `name` WHEN 'merge_status' THEN `info` ELSE '' END ) `merge_status`,                 MAX(CASE `name` WHEN 'broadcast_version' THEN `value` ELSE 0 END ) `broadcast_version`,       MAX(CASE `name` WHEN 'all_merged_version' THEN `value` ELSE 0 END ) `all_merged_version`,     MAX(CASE `name` WHEN 'last_merged_version' THEN `value` ELSE 0 END ) `last_merged_version`,   MAX(CASE `name` WHEN 'merge_start_time' THEN `value` ELSE 0 END ) `merge_start_time`,         MAX(CASE `name` WHEN 'last_merged_time' THEN `value` ELSE 0 END ) `last_merged_time`,         MAX(CASE `name` WHEN 'is_merge_timeout' THEN `value` ELSE 0 END ) `merge_timeout`            FROM oceanbase.__all_zone WHERE `zone` <> '' GROUP BY `zone`
2022-11-05 23:37:46.330  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SELECT                                      zone, gmt_create, gmt_modified, svr_ip, svr_port, inner_port, with_rootserver,                UPPER(`status`) as `status`, build_version, stop_time, start_service_time, last_offline_time,  block_migrate_in_time, id FROM oceanbase.__all_server ORDER BY zone ASC, id ASC
2022-11-05 23:37:46.362  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.364  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.373  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SELECT /*+read_consistency(weak) */  svr_ip, svr_port, zone, leader_count, unit_num,                                cpu_total, cpu_assigned, cpu_assigned_percent,                                 mem_total, mem_assigned, mem_assigned_percent,                                 disk_total, disk_assigned, disk_assigned_percent,                              disk_in_use, round(disk_in_use * 100 / disk_total) disk_used_percent FROM oceanbase.__all_virtual_server_stat
2022-11-05 23:37:46.442  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.445  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.445  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SHOW PARAMETERS WHERE scope = 'cluster' AND name LIKE ? , args: [system_memory]
2022-11-05 23:37:46.483  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SHOW PARAMETERS WHERE scope = 'cluster' AND name LIKE ? , args: [system_memory_percentage]
2022-11-05 23:37:46.532  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.533  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.539  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SELECT /*+read_consistency(weak) */  svr_ip, svr_port, total_size, free_size, total_size - free_size AS used_size,       ROUND((total_size-free_size) / total_size * 100, 0) AS used_percent     FROM oceanbase.__all_virtual_disk_stat
2022-11-05 23:37:46.593  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.595  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.596  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SHOW PARAMETERS WHERE scope = 'cluster' AND name LIKE ? , args: [obconfig_url]
2022-11-05 23:37:46.670  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.673  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.678  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SELECT  /*+ READ_CONSISTENCY(WEAK) QUERY_TIMEOUT(60000000) */ t1.unit_id, t1.resource_pool_id, t1.zone, t1.svr_ip, t1.svr_port, t1.`status`, t1.replica_type,  t2.tenant_id, t3.tenant_name, t2.`name` AS resource_pool_name, t1.migrate_from_svr_ip, t1.migrate_from_svr_port, t1.manual_migrate FROM oceanbase.__all_unit AS t1 JOIN oceanbase.__all_resource_pool AS t2 ON t1.resource_pool_id = t2.resource_pool_id JOIN oceanbase.__all_tenant t3 ON t2.tenant_id = t3.tenant_id WHERE 1=1, args: []
2022-11-05 23:37:46.721  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.723  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.723  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: SELECT  cluster_id, cluster_name, created, cluster_role, cluster_status, current_scn,  standby_became_primary_scn, primary_cluster_id, (time_to_usec(now(6)) - current_scn)/1000000 as delay_time, protection_mode, redo_transport_options, protection_level FROM oceanbase.v$ob_cluster
2022-11-05 23:37:46.760  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : get credential from obsdk context, userId=200,  clusterName=lx_test, tenantName=sys, dbUser=ocp_monitor
2022-11-05 23:37:46.762  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.s.o.o.f.ConnectPropertiesBuilder   : user: ocp_monitor, tenant: sys, clusterName:lx_test, obClusterId:22, connectionMode: proxy
2022-11-05 23:37:46.762  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: select max(value) value from oceanbase.__all_virtual_sys_parameter_stat where name = 'min_observer_version'
2022-11-05 23:37:46.789  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.c.obsdk.connector.ConnectTemplate  : [obsdk] sql: select zone,svr_type,svr_ip,svr_port,name,data_type,value,info,section,scope,source,edit_level from __all_virtual_sys_parameter_stat where scope = 'cluster' and name like ?, args: [_enable_oracle_priv_check]
2022-11-05 23:37:46.815  INFO 52 --- [http-nio-8080-exec-1232,e1015c3362ef45a7,c20568572287] c.a.o.server.trace.RequestTracingAspect  : API OK: [GET /api/v2/ob/clusters/4000001 client=172.22.169.13, traceId=e1015c3362ef45a7, duration=1613 ms]
``` 

一共发了11组SQL:

  - __all_zone
    - SELECT `zone`, MAX(CASE `name` WHEN 'region' THEN `info` ELSE '' END ) `region`, MAX(CASE `name` WHEN 'idc' THEN `info` ELSE '' END ) `idc`, MAX(CASE `name` WHEN 'status' THEN `info` ELSE '' END ) `status`, MAX(CASE `name` WHEN 'merge_status' THEN `info` ELSE '' END ) `merge_status`, MAX(CASE `name` WHEN 'broadcast_version' THEN `value` ELSE 0 END ) `broadcast_version`, MAX(CASE `name` WHEN 'all_merged_version' THEN `value` ELSE 0 END ) `all_merged_version`, MAX(CASE `name` WHEN 'last_merged_version' THEN `value` ELSE 0 END ) `last_merged_version`, MAX(CASE `name` WHEN 'merge_start_time' THEN `value` ELSE 0 END ) `merge_start_time`, MAX(CASE `name` WHEN 'last_merged_time' THEN `value` ELSE 0 END ) `last_merged_time`, MAX(CASE `name` WHEN 'is_merge_timeout' THEN `value` ELSE 0 END ) `merge_timeout` FROM oceanbase.__all_zone WHERE `zone` <> '' GROUP BY `zone`
  - __all_server
    - SELECT zone, gmt_create, gmt_modified, svr_ip, svr_port, inner_port, with_rootserver, UPPER(`status`) as `status`, build_version, stop_time, start_service_time, last_offline_time, block_migrate_in_time, id FROM oceanbase.__all_server ORDER BY zone ASC, id ASC
  - __all_virtual_server_stat
    - SELECT /*+read_consistency(weak) */ svr_ip, svr_port, zone, leader_count, unit_num, cpu_total, cpu_assigned, cpu_assigned_percent, mem_total, mem_assigned, mem_assigned_percent, disk_total, disk_assigned, disk_assigned_percent, disk_in_use, round(disk_in_use * 100 / disk_total) disk_used_percent FROM oceanbase.__all_virtual_server_stat
  - system_memory
    - SHOW PARAMETERS WHERE scope = 'cluster' AND name LIKE ? , args: [system_memory]
  - system_memory_percentage
    - SHOW PARAMETERS WHERE scope = 'cluster' AND name LIKE ? , args: [system_memory_percentage]
  - __all_virtual_disk_stat
    - SELECT /*+read_consistency(weak) */ svr_ip, svr_port, total_size, free_size, total_size - free_size AS used_size, ROUND((total_size-free_size) / total_size * 100, 0) AS used_percent FROM oceanbase.__all_virtual_disk_stat
  - obconfig_url
    - SHOW PARAMETERS WHERE scope = 'cluster' AND name LIKE ? , args: [obconfig_url]
  - __all_resource_pool
    - SELECT /*+ READ_CONSISTENCY(WEAK) QUERY_TIMEOUT(60000000) */ t1.unit_id, t1.resource_pool_id, t1.zone, t1.svr_ip, t1.svr_port, t1.`status`, t1.replica_type, t2.tenant_id, t3.tenant_name, t2.`name` AS resource_pool_name, t1.migrate_from_svr_ip, t1.migrate_from_svr_port, t1.manual_migrate FROM oceanbase.__all_unit AS t1 JOIN oceanbase.__all_resource_pool AS t2 ON t1.resource_pool_id = t2.resource_pool_id JOIN oceanbase.__all_tenant t3 ON t2.tenant_id = t3.tenant_id WHERE 1=1, args: []
  - v$ob_cluster
    - SELECT cluster_id, cluster_name, created, cluster_role, cluster_status, current_scn, standby_became_primary_scn, primary_cluster_id, (time_to_usec(now(6)) - current_scn)/1000000 as delay_time, protection_mode, redo_transport_options, protection_level FROM oceanbase.v$ob_cluster
  - min_observer_version
    - select max(value) value from oceanbase.__all_virtual_sys_parameter_stat where name = 'min_observer_version'
  - _enable_oracle_priv_check
    - select zone,svr_type,svr_ip,svr_port,name,data_type,value,info,section,scope,source,edit_level from __all_virtual_sys_parameter_stat where scope = 'cluster' and name like ?, args: [_enable_oracle_priv_check]

  
连接 lx_test 的 租户的root@sys账号, 密码通过OCP界面直接修改: 

```
mysql -h10.186.60.81 -P2881 -uroot@sys -p
``` 

查询上述11个SQL, 没有与server的starting状态和operation状态相关的字段

换个思路: 查看OCP元数据库

```
mysql -h10.186.60.34 -P2881 -uroot@ocp_meta -p
``` 

查看ob_server表

![image2022-11-8 16:45:57.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2016%3A45%3A57.png)

将状态修改成RUNNING, 一段时间后, OCP并未修正状态

猜测OCP处理逻辑如下, (zone和cluster上的状态同理): 

  1. 从 UI 启动observer, 会将OCP元数据库里的该节点状态更新为 "STARTING"
  2. OCP监听 OB集群的状态 (通过状态SQL), 如果节点加入集群, 将状态更新为 "RUNNING"
  3. 否则, 比如observer启动中崩溃, 节点状态将一直维持为 "STARTING", UI上无法进行其他运维操作

建议: 

  - 在UI上提供observer状态检查, 如果进程崩溃, 允许进一步的运维操作和日志查看
  - 在处理逻辑3中, 增加探测逻辑, 通过ocp-agent探测: 如果observer进程消失, 则设置状态为停止或故障

# 诊断-2

通过界面重启observer, 未见进程

查询ocp-agent日志, 在pos_proxy.log中找到被调用的命令, 日志报错如下: 

```
2022-11-08 09:46:42,817 - INFO - 18413:139638870120256 - pos_proxy:68 - call pos method run_cmd with token=b62576c840c3fcc131ef2b63dbbbed0c and args: (u'admin', u'cd /home/admi
n/oceanbase && /home/admin/oceanbase/bin/observer') started
2022-11-08 09:46:43,482 - INFO - 18413:139638870120256 - pos_proxy:71 - call pos method run_cmd with token=b62576c840c3fcc131ef2b63dbbbed0c and args: (u'admin', u'cd /home/admi
n/oceanbase && /home/admin/oceanbase/bin/observer') got result: /home/admin/oceanbase/bin/observer
[2022-11-08 09:46:43.465097] ERROR [LIB] pidfile_test (utility.cpp:1175) [24160][0][Y0-0000000000000000] [lt=0] fid file doesn't exist(pidfile="run/observer.pid") BACKTRACE:0x9
e781da 0x9dbadb0 0x2c4bf8d 0x2ccf8db 0x9e7d85f 0x2c54ec8 0x7fcb645fd555 0x2c4884d
2022-11-08 09:47:03,576 - INFO - 18413:139638870120256 - pos_proxy:68 - call pos method run_cmd with token=2888cd28d589008b7963acbbcb7496f9 and args: (u'root', u'pgrep -c obser
ver | xargs --no-run-if-empty') started
``` 

解开堆栈: 

```
[root@localhost agent]# addr2line -pCfe /home/admin/oceanbase/bin/observer 0x9e781da 0x9dbadb0 0x2c4bf8d 0x2ccf8db 0x9e7d85f 0x2c54ec8 0x7fcb645fd555 0x2c4884d
oceanbase::common::lbt() at ??:?
oceanbase::common::ObLogger::log_data(char const*, int, oceanbase::common::ObLogger::LogLocation, oceanbase::common::ObLogger::LogBuffer&) at ??:?
oceanbase::common::ObLogger::log_message_kv(char const*, int, char const*, int, char const*, unsigned long, char const*, char const*) at main.cpp:?
void oceanbase::common::OB_PRINT<char const*>(char const*, int, char const*, int, char const*, unsigned long, char const*, char const*, char const* const&) at ??:?
oceanbase::common::start_daemon(char const*) at ??:?
main at ??:?
?? ??:0
_start at ??:?
``` 

该行错误日志在 <https://github.com/oceanbase/oceanbase/issues/137> 修复, 不会影响启动流程

使用strace跟踪pos_proxy:

![image2022-11-8 18:49:35.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2018%3A49%3A35.png)

触发界面操作

找到启动observer的线程: 16522:

![image2022-11-8 18:50:41.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2018%3A50%3A41.png)

该线程在打印 fid file doesn't exist报错后, 生出了线程16585

![image2022-11-8 18:51:19.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2018%3A51%3A19.png)

找到线程16585: 

![image2022-11-8 18:51:53.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2018%3A51%3A53.png)

报错为 can't open pid file (改进点: 该日志未出现在OCP日志中)

找到该线程对pid文件的访问: 

![image2022-11-8 18:52:58.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2018%3A52%3A58.png)

判断 16585 的用户 对 run/observer.pid 没有权限, 查找 16585的父线程和祖父线程: 

16585 - 16522 - 16472

![image2022-11-8 19:5:45.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2019%3A5%3A45.png)

可以找到 16472切换了运行用户至 500 (admin): 

![image2022-11-8 19:6:39.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2019%3A6%3A39.png)

找到16472的父线程 16468:

![image2022-11-8 19:7:34.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2019%3A7%3A34.png)

其通过runuser切换了运行用户:

![image2022-11-8 19:8:56.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-8%2019%3A8%3A56.png)

执行了一个临时脚本

问题解决: 将oceanbase的目录整体刷一次owner:

```
chown -R admin: .
``` 

# 问题-1

部分observer节点的日志被删除, 怀疑是人工失误

10.186.60.81

![image2022-11-6 0:20:57.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-6%200%3A20%3A57.png)

# 问题-2

10.186.60.81

磁盘路径不合理

![image2022-11-6 0:21:46.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-6%200%3A21%3A46.png)

# 改进建议-1 

![image2022-11-5 23:8:19.png](/assets/01KJBYXQTFVXZDQBG4A4V8EBCQ/image2022-11-5%2023%3A8%3A19.png)

observer 启动失败的原因, 未出现在OCP日志中, pidfile_test太过抽象
