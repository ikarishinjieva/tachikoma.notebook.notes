---
title: 20221109 - OMA评估
confluence_page_id: 2130070
created_at: 2022-11-09T03:44:04+00:00
updated_at: 2022-11-15T10:49:19+00:00
---

# 评估confluence库

```
sh bin/start.sh \
--name test \
--mode ANALYZE \
--from-type DB \
--evaluate-mode SOURCE_TARGET \
--source-db-type MYSQL \
--source-db-version 5.7 \
--source-db-host 10.186.62.73 \
--source-db-port 5729 \
--source-db-user a \
--source-db-password 12344321 \
--schemas "confluence" \
--target-db-type OBMYSQL \
--target-db-version 2.2.x
``` 

开启general log: 

```
root@ubuntu:~/sandboxes/confluence_db/data# cat ubuntu.log
/root/opt/mysql/5.7.29/bin/mysqld, Version: 5.7.29-debug (MySQL Community Server - Debug (GPL)). started with:
Tcp port: 5729  Unix socket: /tmp/mysql_sandbox5729.sock
Time                 Id Command    Argument
2022-11-09T02:58:10.454942Z	   17 Quit
2022-11-09T02:58:58.621576Z	   18 Connect	a@10.186.62.73 on confluence using TCP/IP
2022-11-09T02:58:58.625251Z	   18 Query	/* mysql-connector-java-8.0.22 (Revision: d64b664fa93e81296a377de031b8123a67e6def2) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@collation_connection AS collation_connection, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_write_timeout AS net_write_timeout, @@performance_schema AS performance_schema, @@query_cache_size AS query_cache_size, @@query_cache_type AS query_cache_type, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@transaction_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
2022-11-09T02:58:58.635909Z	   18 Query	SHOW WARNINGS
2022-11-09T02:58:58.642435Z	   18 Query	SET NAMES utf8mb4
2022-11-09T02:58:58.642937Z	   18 Query	SET character_set_results = NULL
2022-11-09T02:58:58.643492Z	   18 Query	SET autocommit=1
2022-11-09T02:58:58.650025Z	   18 Query	SELECT @@session.transaction_read_only
2022-11-09T02:58:58.650535Z	   18 Query	SELECT @@session.transaction_isolation
2022-11-09T02:59:08.663833Z	   19 Connect	a@10.186.62.73 on confluence using TCP/IP
2022-11-09T02:59:08.664796Z	   19 Query	/* mysql-connector-java-8.0.22 (Revision: d64b664fa93e81296a377de031b8123a67e6def2) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@collation_connection AS collation_connection, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_write_timeout AS net_write_timeout, @@performance_schema AS performance_schema, @@query_cache_size AS query_cache_size, @@query_cache_type AS query_cache_type, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@transaction_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
2022-11-09T02:59:08.665546Z	   19 Query	SHOW WARNINGS
2022-11-09T02:59:08.666098Z	   19 Query	SET NAMES utf8mb4
2022-11-09T02:59:08.666282Z	   19 Query	SET character_set_results = NULL
2022-11-09T02:59:08.666558Z	   19 Query	SET autocommit=1
2022-11-09T02:59:08.667461Z	   19 Query	SELECT @@session.transaction_read_only
2022-11-09T02:59:08.667789Z	   19 Query	SELECT @@session.transaction_isolation
2022-11-09T02:59:18.681075Z	   20 Connect	a@10.186.62.73 on confluence using TCP/IP
2022-11-09T02:59:18.682097Z	   20 Query	/* mysql-connector-java-8.0.22 (Revision: d64b664fa93e81296a377de031b8123a67e6def2) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@collation_connection AS collation_connection, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_write_timeout AS net_write_timeout, @@performance_schema AS performance_schema, @@query_cache_size AS query_cache_size, @@query_cache_type AS query_cache_type, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@transaction_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
2022-11-09T02:59:18.682796Z	   20 Query	SHOW WARNINGS
2022-11-09T02:59:18.683355Z	   20 Query	SET NAMES utf8mb4
2022-11-09T02:59:18.683785Z	   20 Query	SET character_set_results = NULL
2022-11-09T02:59:18.684134Z	   20 Query	SET autocommit=1
2022-11-09T02:59:18.685203Z	   20 Query	SELECT @@session.transaction_read_only
2022-11-09T02:59:18.685756Z	   20 Query	SELECT @@session.transaction_isolation
2022-11-09T02:59:28.712425Z	   21 Connect	a@10.186.62.73 on confluence using TCP/IP
2022-11-09T02:59:28.713744Z	   21 Query	/* mysql-connector-java-8.0.22 (Revision: d64b664fa93e81296a377de031b8123a67e6def2) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@collation_connection AS collation_connection, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_write_timeout AS net_write_timeout, @@performance_schema AS performance_schema, @@query_cache_size AS query_cache_size, @@query_cache_type AS query_cache_type, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@transaction_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
2022-11-09T02:59:28.714623Z	   21 Query	SHOW WARNINGS
2022-11-09T02:59:28.715437Z	   21 Query	SET NAMES utf8mb4
2022-11-09T02:59:28.715669Z	   21 Query	SET character_set_results = NULL
2022-11-09T02:59:28.715938Z	   21 Query	SET autocommit=1
2022-11-09T02:59:28.716681Z	   21 Query	SELECT @@session.transaction_read_only
2022-11-09T02:59:28.717004Z	   21 Query	SELECT @@session.transaction_isolation
2022-11-09T02:59:38.731907Z	   22 Connect	a@10.186.62.73 on confluence using TCP/IP
2022-11-09T02:59:38.732922Z	   22 Query	/* mysql-connector-java-8.0.22 (Revision: d64b664fa93e81296a377de031b8123a67e6def2) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@collation_connection AS collation_connection, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_write_timeout AS net_write_timeout, @@performance_schema AS performance_schema, @@query_cache_size AS query_cache_size, @@query_cache_type AS query_cache_type, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@transaction_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
2022-11-09T02:59:38.733666Z	   22 Query	SHOW WARNINGS
2022-11-09T02:59:38.734310Z	   22 Query	SET NAMES utf8mb4
2022-11-09T02:59:38.734525Z	   22 Query	SET character_set_results = NULL
2022-11-09T02:59:38.734797Z	   22 Query	SET autocommit=1
2022-11-09T02:59:38.735521Z	   22 Query	SELECT @@session.transaction_read_only
2022-11-09T02:59:38.735823Z	   22 Query	SELECT @@session.transaction_isolation
2022-11-09T02:59:38.792667Z	   22 Query	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'confluence' AND  TABLE_TYPE = 'BASE TABLE'
2022-11-09T02:59:38.810936Z	   22 Query	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'confluence' AND  TABLE_TYPE = 'VIEW'
2022-11-09T02:59:38.817232Z	   22 Query	SELECT TRIGGER_NAME FROM INFORMATION_SCHEMA.TRIGGERS WHERE TRIGGER_SCHEMA = 'confluence'
2022-11-09T02:59:38.838740Z	   22 Query	SELECT ROUTINE_NAME FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'confluence' AND ROUTINE_TYPE= 'FUNCTION'
2022-11-09T02:59:38.842665Z	   22 Query	SELECT ROUTINE_NAME FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'confluence' AND ROUTINE_TYPE= 'PROCEDURE'
2022-11-09T02:59:38.845165Z	   22 Query	SELECT EVENT_NAME FROM INFORMATION_SCHEMA.EVENTS WHERE EVENT_SCHEMA = 'confluence'
2022-11-09T02:59:38.848672Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_187ccc_sidebar_link`
2022-11-09T02:59:38.856050Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_21d670_whitelist_rules`
2022-11-09T02:59:38.857591Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_26db7f_entities_to_room_cfg`
2022-11-09T02:59:38.859043Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_26db7f_entities_to_rooms`
2022-11-09T02:59:38.860905Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_38321b_custom_content_link`
2022-11-09T02:59:38.862789Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_42e351_health_check_entity`
2022-11-09T02:59:38.864190Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_54c900_c_template_ref`
2022-11-09T02:59:38.865610Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_54c900_content_blueprint_ao`
2022-11-09T02:59:38.866761Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_54c900_space_blueprint_ao`
2022-11-09T02:59:38.867929Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_5f3884_feature_discovery`
2022-11-09T02:59:38.868948Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_5fb9d7_aohip_chat_link`
2022-11-09T02:59:38.870216Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_5fb9d7_aohip_chat_user`
2022-11-09T02:59:38.871631Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_6384ab_discovered`
2022-11-09T02:59:38.872817Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_6384ab_feature_metadata_ao`
2022-11-09T02:59:38.876803Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_7cde43_event`
2022-11-09T02:59:38.878259Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_7cde43_filter_param`
2022-11-09T02:59:38.879854Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_7cde43_notification`
2022-11-09T02:59:38.881481Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_7cde43_notification_scheme`
2022-11-09T02:59:38.882942Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_7cde43_recipient`
2022-11-09T02:59:38.884184Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_7cde43_server_config`
2022-11-09T02:59:38.885427Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_7cde43_server_param`
2022-11-09T02:59:38.886668Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_88bb94_batch_notification`
2022-11-09T02:59:38.887869Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_92296b_aorecently_viewed`
2022-11-09T02:59:38.889270Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_9412a1_aonotification`
2022-11-09T02:59:38.892695Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_9412a1_aoregistration`
2022-11-09T02:59:38.894028Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_9412a1_aotask`
2022-11-09T02:59:38.895346Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_9412a1_aouser`
2022-11-09T02:59:38.896334Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_9412a1_user_app_link`
2022-11-09T02:59:38.897318Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_a0b856_web_hook_listener_ao`
2022-11-09T02:59:38.898260Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_baf3aa_aoinline_task`
2022-11-09T02:59:38.899481Z	   22 Query	SHOW CREATE TABLE `confluence`.`ao_dc98ae_aohelp_tip`
2022-11-09T02:59:38.900435Z	   22 Query	SHOW CREATE TABLE `confluence`.`attachmentdata`
2022-11-09T02:59:38.901354Z	   22 Query	SHOW CREATE TABLE `confluence`.`bandana`
2022-11-09T02:59:38.902273Z	   22 Query	SHOW CREATE TABLE `confluence`.`bodycontent`
2022-11-09T02:59:38.904477Z	   22 Query	SHOW CREATE TABLE `confluence`.`clustersafety`
2022-11-09T02:59:38.905472Z	   22 Query	SHOW CREATE TABLE `confluence`.`confancestors`
2022-11-09T02:59:38.906362Z	   22 Query	SHOW CREATE TABLE `confluence`.`confversion`
2022-11-09T02:59:38.907265Z	   22 Query	SHOW CREATE TABLE `confluence`.`content`
2022-11-09T02:59:38.909540Z	   22 Query	SHOW CREATE TABLE `confluence`.`content_label`
2022-11-09T02:59:38.910580Z	   22 Query	SHOW CREATE TABLE `confluence`.`content_perm`
2022-11-09T02:59:38.911636Z	   22 Query	SHOW CREATE TABLE `confluence`.`content_perm_set`
2022-11-09T02:59:38.912544Z	   22 Query	SHOW CREATE TABLE `confluence`.`content_relation`
2022-11-09T02:59:38.913713Z	   22 Query	SHOW CREATE TABLE `confluence`.`contentproperties`
2022-11-09T02:59:38.914887Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_app_dir_group_mapping`
2022-11-09T02:59:38.915900Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_app_dir_mapping`
2022-11-09T02:59:38.916885Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_app_dir_operation`
2022-11-09T02:59:38.917835Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_application`
2022-11-09T02:59:38.918871Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_application_address`
2022-11-09T02:59:38.919760Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_application_attribute`
2022-11-09T02:59:38.920647Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_directory`
2022-11-09T02:59:38.921647Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_directory_attribute`
2022-11-09T02:59:38.922563Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_directory_operation`
2022-11-09T02:59:38.923418Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_group`
2022-11-09T02:59:38.924379Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_group_attribute`
2022-11-09T02:59:38.925280Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_membership`
2022-11-09T02:59:38.926309Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_user`
2022-11-09T02:59:38.927388Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_user_attribute`
2022-11-09T02:59:38.928370Z	   22 Query	SHOW CREATE TABLE `confluence`.`cwd_user_credential_record`
2022-11-09T02:59:38.929250Z	   22 Query	SHOW CREATE TABLE `confluence`.`decorator`
2022-11-09T02:59:38.930145Z	   22 Query	SHOW CREATE TABLE `confluence`.`external_entities`
2022-11-09T02:59:38.931011Z	   22 Query	SHOW CREATE TABLE `confluence`.`external_members`
2022-11-09T02:59:38.931935Z	   22 Query	SHOW CREATE TABLE `confluence`.`extrnlnks`
2022-11-09T02:59:38.932897Z	   22 Query	SHOW CREATE TABLE `confluence`.`follow_connections`
2022-11-09T02:59:38.933835Z	   22 Query	SHOW CREATE TABLE `confluence`.`groups`
2022-11-09T02:59:38.934710Z	   22 Query	SHOW CREATE TABLE `confluence`.`hibernate_unique_key`
2022-11-09T02:59:38.935504Z	   22 Query	SHOW CREATE TABLE `confluence`.`imagedetails`
2022-11-09T02:59:38.936324Z	   22 Query	SHOW CREATE TABLE `confluence`.`indexqueueentries`
2022-11-09T02:59:38.937128Z	   22 Query	SHOW CREATE TABLE `confluence`.`journalentry`
2022-11-09T02:59:38.938139Z	   22 Query	SHOW CREATE TABLE `confluence`.`keystore`
2022-11-09T02:59:38.938926Z	   22 Query	SHOW CREATE TABLE `confluence`.`label`
2022-11-09T02:59:38.939736Z	   22 Query	SHOW CREATE TABLE `confluence`.`likes`
2022-11-09T02:59:38.940582Z	   22 Query	SHOW CREATE TABLE `confluence`.`links`
2022-11-09T02:59:38.941714Z	   22 Query	SHOW CREATE TABLE `confluence`.`local_members`
2022-11-09T02:59:38.942560Z	   22 Query	SHOW CREATE TABLE `confluence`.`logininfo`
2022-11-09T02:59:38.943486Z	   22 Query	SHOW CREATE TABLE `confluence`.`notifications`
2022-11-09T02:59:38.944556Z	   22 Query	SHOW CREATE TABLE `confluence`.`os_group`
2022-11-09T02:59:38.945435Z	   22 Query	SHOW CREATE TABLE `confluence`.`os_propertyentry`
2022-11-09T02:59:38.946519Z	   22 Query	SHOW CREATE TABLE `confluence`.`os_user`
2022-11-09T02:59:38.947406Z	   22 Query	SHOW CREATE TABLE `confluence`.`os_user_group`
2022-11-09T02:59:38.948417Z	   22 Query	SHOW CREATE TABLE `confluence`.`pagetemplates`
2022-11-09T02:59:38.949511Z	   22 Query	SHOW CREATE TABLE `confluence`.`plugindata`
2022-11-09T02:59:38.951096Z	   22 Query	SHOW CREATE TABLE `confluence`.`remembermetoken`
2022-11-09T02:59:38.952625Z	   22 Query	SHOW CREATE TABLE `confluence`.`spacepermissions`
2022-11-09T02:59:38.953608Z	   22 Query	SHOW CREATE TABLE `confluence`.`spaces`
2022-11-09T02:59:38.954611Z	   22 Query	SHOW CREATE TABLE `confluence`.`trackbacklinks`
2022-11-09T02:59:38.955894Z	   22 Query	SHOW CREATE TABLE `confluence`.`trustedapp`
2022-11-09T02:59:38.956883Z	   22 Query	SHOW CREATE TABLE `confluence`.`trustedapprestriction`
2022-11-09T02:59:38.957738Z	   22 Query	SHOW CREATE TABLE `confluence`.`user_mapping`
2022-11-09T02:59:38.958574Z	   22 Query	SHOW CREATE TABLE `confluence`.`user_relation`
2022-11-09T02:59:38.959449Z	   22 Query	SHOW CREATE TABLE `confluence`.`usercontent_relation`
2022-11-09T02:59:38.960374Z	   22 Query	SHOW CREATE TABLE `confluence`.`users`
root@ubuntu:~/sandboxes/confluence_db/data#
``` 

评估的对象: 

```
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'confluence' AND  TABLE_TYPE = 'BASE TABLE'
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'confluence' AND  TABLE_TYPE = 'VIEW'
SELECT TRIGGER_NAME FROM INFORMATION_SCHEMA.TRIGGERS WHERE TRIGGER_SCHEMA = 'confluence'
SELECT ROUTINE_NAME FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'confluence' AND ROUTINE_TYPE= 'FUNCTION'
SELECT ROUTINE_NAME FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'confluence' AND ROUTINE_TYPE= 'PROCEDURE'
SELECT EVENT_NAME FROM INFORMATION_SCHEMA.EVENTS WHERE EVENT_SCHEMA = 'confluence'
``` 

  - table
  - view
  - trigger
  - function
  - procedure
  - event

调整 config/logback.xml 日志级别, 获取DEBUG日志:

[附件: 1.log] 

根据日志, 其代码位置位于: 

![image2022-11-9 19:2:3.png](/assets/01KJBYSTE6KE2T030VC4M9TP90/image2022-11-9%2019%3A2%3A3.png)

反编译OMA的jar包

  - AbstractEvaluate.java #evalSingleSql, 其进行几部分检查
    - 按照OB的MySQL parser, 进行 Grammar/Syntax 检查
    - 一些Rule, 位于: mysql/GeneralSqlChecker.java
      - Column
      - Function
      - Normal SQL
    - Rule的配置列表 是通过 CheckRuleBuilder 进行构建, 在配置列表中的Rule才会被检查

# 发现缺陷

CheckRuleBuilder.mysqlBuild 中, 仅返回了 obMysql323CheckRule (与3.2.3相关的Rule), 目标版本配置为其他版本时, Rule均不会生效

建议修正代码或修正文档: 

![image2022-11-10 11:13:48.png](/assets/01KJBYSTE6KE2T030VC4M9TP90/image2022-11-10%2011%3A13%3A48.png)

# 对规则集的猜测

猜测规则是由 OBMYSQL3230.incmpt 控制:

![image2022-11-10 11:28:23.png](/assets/01KJBYSTE6KE2T030VC4M9TP90/image2022-11-10%2011%3A28%3A23.png)

但代码上并不支持: CheckRuleBuilder.DbTypeToVerifyResultsFile 初始化为空, 查找不到规则文件. 怀疑是反编译的偏差

# 对SQL进行评估-文本文件

SQL文本: 

```
select biz_param_code , biz_param_val , biz_param_chn  FROM biz_parameter WHERE biz_param_code = 'b0';
$$
select biz_param_code , biz_param_val , biz_param_chn INTO :b0 , :b1 , :b2 FROM biz_parameter WHERE biz_param_code = :b0
$$
select biz_param_code , biz_param_val , biz_param_chn  FROM biz_parameter WHERE biz_param_code = 'b0';
$$

``` 

OMA命令: 

```
sh bin/start.sh \
--name test \
--mode ANALYZE \
--from-type TEXT \
--evaluate-mode SOURCE_TARGET \
--source-file "/tmp/10.txt" \
--source-db-type MYSQL \
--source-db-version 5.7 \
--schemas DEFAULT \
--target-db-type OBMYSQL \
--target-db-version 3.2.3 \
--process-thread-count 5

``` 

报告: 

[附件: test_20221110_114905-basic.html] 

# 对于mode的解释

\--mode值的定义, 在common.jar中Content.java

```
   DUMP("DUMP"),
/* 118 */     ANALYZE("ANALYZE"),
/* 119 */     EVALUATE("EVALUATE"),
/* 120 */     CONSOLE("CONSOLE"),
/* 121 */     REPLAY("REPLAY"),
/* 122 */     ANALYZE_TOTAL("ANALYZE_TOTAL"),
/* 123 */     WORKER("WORKER"),
/* 124 */     PORTRAIT("PORTRAIT");
``` 

代码位: command.jar, Processor.processCommand

  - ANALYZE: 兼容性分析: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486591>
    - 函数: scheduleAssessmentTask
  - ANALYZE_TOTAL: 整库评估: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486605>
    - 函数: assessmentAllModeTask
  - REPLAY: 性能评估/回放: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486598>
    - 函数: replayTask (单机版), replayTaskWithIgnite (分布式版)
  - WORKER: 用于分布式回放: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486601>
  - DUMP: 导出 OceanBase 数据库的 SQL 语句: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486594>
    - 函数: dumpOceanBase
  - PORTRAIT:
    - 函数: analyzeDbPortrait
  - CONSOLE: 
    - 函数: runWithConsole
    - ![image2022-11-11 0:9:35.png](/assets/01KJBYSTE6KE2T030VC4M9TP90/image2022-11-11%200%3A9%3A35.png)
  - EVALUATE
    - 没有对应操作函数, 怀疑是bug

### PORTRAIT模式

代码位: 找到getDbPortraitData的实现类: MySQL相关包括: 

![image2022-11-11 0:14:5.png](/assets/01KJBYSTE6KE2T030VC4M9TP90/image2022-11-11%200%3A14%3A5.png)

即获取数据库的画像信息

```
sh bin/start.sh --name test --mode PORTRAIT --from-type DB --evaluate-mode SOURCE_TARGET --source-db-type MYSQL --source-db-version 5.7 --source-db-host 10.186.62.73 --source-db-port 5729 --source-db-user a --source-db-password 12344321 --schemas "confluence" --target-db-type OBMYSQL --target-db-version 3.2.3
``` 

会出现报错: 

![image2022-11-11 0:33:22.png](/assets/01KJBYSTE6KE2T030VC4M9TP90/image2022-11-11%200%3A33%3A22.png)

判断代码有误, 没有进行本地sqlite的初始化 (参考 ScheduleServiceImpl 中调用configureLocalDataSource 的实现)

# 各参数的尝试

ANALYZE模式

```
sh bin/start.sh --name test --mode ANALYZE --from-type DB --evaluate-mode SOURCE_TARGET --source-db-type MYSQL --source-db-version 5.7 --source-db-host 10.186.62.73 --source-db-port 5729 --source-db-user a --source-db-password 12344321 --schemas "confluence" --target-db-type OBMYSQL --target-db-version 3.2.3
``` 

评估SQL文件

```
sh bin/start.sh --name test --mode ANALYZE --evaluate-mode SOURCE_TARGET --source-db-type MYSQL --source-db-version 5.7 --source-db-host 10.186.62.73 --source-db-port 5729 --source-db-user a --source-db-password 12344321 --schemas "confluence" --target-db-type OBMYSQL --target-db-version 3.2.3 --from-type TEXT --source-file /tmp/1.sql
 
---
 
cat /tmp/1.sql
 
select biz_param_code , biz_param_val , biz_param_chn  FROM biz_parameter WHERE biz_param_code = 'b0';
$$
select biz_param_code , biz_param_val , biz_param_chn INTO :b0 , :b1 , :b2 FROM biz_parameter WHERE biz_param_code = :b0
$$
select biz_param_code , biz_param_val , biz_param_chn  FROM biz_parameter WHERE biz_param_code = 'b0';
$$
``` 

sql-delimiter参数

```
sh bin/start.sh --name test --mode ANALYZE --evaluate-mode SOURCE_TARGET --source-db-type MYSQL --source-db-version 5.7 --source-db-host 10.186.62.73 --source-db-port 5729 --source-db-user a --source-db-password 12344321 --schemas "confluence" --target-db-type OBMYSQL --target-db-version 3.2.3 --from-type TEXT --source-file /tmp/1.sql --sql-delimiter '//'

---

cat /tmp/1.sql

select biz_param_code , biz_param_val , biz_param_chn  FROM biz_parameter WHERE biz_param_code = 'b0';
//
select biz_param_code , biz_param_val , biz_param_chn INTO :b0 , :b1 , :b2 FROM biz_parameter WHERE biz_param_code = :b0
//
select biz_param_code , biz_param_val , biz_param_chn  FROM biz_parameter WHERE biz_param_code = 'b0';
//
```
