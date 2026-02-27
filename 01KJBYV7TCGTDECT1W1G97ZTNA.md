---
title: 20221117 - OMS试用
confluence_page_id: 2130172
created_at: 2022-11-17T08:04:46+00:00
updated_at: 2022-11-22T02:48:43+00:00
---

# 安装

docker load -i oms.feature_3.3.0-bp1.202207050628.tar.gz

image名: [reg.docker.alibaba-inc.com/oboms/oms-all-in-one:feature_3.3.0-bp1](<http://reg.docker.alibaba-inc.com/oboms/oms-all-in-one:feature_3.3.0-bp1>)

配置文件: 

```
oms_meta_host: 10.186.62.73
oms_meta_port: 5730
oms_meta_user: test
oms_meta_password: test
drc_rm_db: oms_rm
drc_cm_db: oms_cm
drc_cm_heartbeat_db: oms_cm_heartbeat
drc_user: oms_drc_user_name
drc_password: 'OceanBase#oms_pass'
cm_url: http://10.186.62.73:8088
cm_location: 100
cm_region: cn-anhui
cm_region_cn: 安徽
cm_is_default: true
cm_nodes:
- 10.186.62.73
tsdb_service: 'INFLUXDB'
tsdb_enabled: false
tsdb_url: '10.10.10.4:8086'
tsdb_username: username
tsdb_password: 123456
``` 

密码: admin/Admin_123

容器创建命令: 

```
docker run -dit --net host \
-v /opt/oms/config.yaml:/home/admin/conf/config.yaml \
-v /opt/oms/oms_logs:/home/admin/logs \
-v /opt/oms/oms_store:/home/ds/store \
-v /opt/oms/oms_run:/home/ds/run \
-e OMS_HOST_IP=10.186.62.73 \
--privileged=true \
--pids-limit -1 \
--ulimit nproc=65535:65535 \
--name oms_test \
reg.docker.alibaba-inc.com/oceanbase/oms:feature_3.3.0-ce
``` 

# 对容器结构的分析

### 入口脚本: /root/docker_cmd.sh

  - 通过python脚本, 初始化配置 /home/admin/conf/config.yaml
  - 启动supervisord, 配置文件: /etc/supervisor/supervisord.conf

### /etc/supervisor/supervisord.conf

```
[unix_http_server]
file=/tmp/supervisor.sock   ; (the path to the socket file)
#chmod=0700                 ;socket文件的mode，默认是0700
#chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid

[inet_http_server]         ;HTTP服务器，提供web管理界面
port=0.0.0.0:8084        ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性

[supervisord]
user=root
logfile=/home/admin/logs/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=500MB       ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=true                ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
#serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
# docker 中使用套接字有可能不能用
serverurl=http://127.0.0.1:8084

[include]
files = /etc/supervisor/conf.d/*.ini
``` 

纳入了 /etc/supervisor/conf.d/*.ini 配置: 

```
[root@ubuntu ~]# ls -alh /etc/supervisor/conf.d/*.ini
-rw-r--r-- 1 root root 285 Jul  5 11:55 /etc/supervisor/conf.d/base.ini
-rw-r--r-- 1 root root 338 Jul  5 11:55 /etc/supervisor/conf.d/drc_cm.ini
-rw-r--r-- 1 root root 397 Jul  5 11:55 /etc/supervisor/conf.d/drc_supervisor.ini
-rw-r--r-- 1 root root 351 Jul  5 11:55 /etc/supervisor/conf.d/oms_console.ini
-rw-r--r-- 1 root root 316 Jul  5 11:55 /etc/supervisor/conf.d/oms_nginx.ini
``` 

### drc_cm服务 / 集群管理服务

```
[program:oms_drc_cm]
directory = /home/ds/cm/package
command = bash /home/admin/conf/command/start_oms_cm.sh
autostart = true
autorestart = true
startsecs = 10
startretries = 0
stderr_logfile = /home/admin/logs/oms_drc_cm_stderr.log
stdout_logfile = /home/admin/logs/oms_drc_cm_stdout.log
user = root
stopasgroup = true
killasgroup = true
``` 

  - 启动脚本: /home/admin/conf/command/start_oms_cm.sh
    - 启动jetty
    - jetty的包路径: /home/ds/cm/package/deployapp/webapps/cm.war
  - 标准日志: /home/admin/logs/oms_drc_cm_std*.log

### drc_supervisor服务

```
[program:oms_drc_supervisor]
directory = /home/ds/supervisor/
environment=OMS_STORE_DEPLOY_DIR=/home/ds/store
command = bash /home/ds/supervisor/service.sh start foreground
stopsignal = QUIT
autostart = true
autorestart = true
startsecs = 10
startretries = 0
stderr_logfile = /home/admin/logs/oms_drc_supervisor_stderr.log
stdout_logfile = /home/admin/logs/oms_drc_supervisor_stdout.log
user = ds
``` 

  - 启动脚本: /home/ds/supervisor/service.sh start foreground
    - jar入口: oms-supervisor.jar
  - 标准日志: /home/admin/logs/oms_drc_supervisor_std*.log

### oms_console服务

```
[program:oms_console]
directory = /home/ds/ghana/resources
command = bash /home/admin/conf/command/start_oms_console.sh
stdout_logfile = /home/admin/logs/oms_console_stdout.log
stderr_logfile = /home/admin/logs/oms_console_stderr.log
user = root
autostart = true
autorestart = true
startsecs = 10
startretries = 0
stopasgroup = true
killasgroup = true
``` 

  - 启动脚本: /home/admin/conf/command/start_oms_console.sh

    - jar入口: /home/ds/ghana/boot/Ghana-endpoint-1.0.0-executable.jar
    - spring配置: /home/ds/ghana/config/application-oms.properties
  - 标准日志: /home/admin/logs/oms_console_std*.log

### oms_nginx服务

```
[program:nginx]
directory = /root
command = bash /home/admin/conf/command/start_nginx.sh
stdout_logfile = /home/admin/logs/oms_nginx_stdout.log
stderr_logfile = /home/admin/logs/oms_nginx_stderr.log
user = root
autostart = true
autorestart = true
startsecs = 10
startretries = 5
stopasgroup = true
killasgroup = true
``` 

  - 启动脚本: /home/admin/conf/command/start_nginx.sh

    - nginx配置: /home/ds/ghana/config/tengine.conf
      - 映射了8090端口到 ghana
  - 标准日志: /home/admin/logs/oms_nginx_std*.log

# 问题1

/home/ds/ghana/config/tengine.conf 配置中的 server_name: 

![image2022-11-17 13:55:44.png](/assets/01KJBYV7TCGTDECT1W1G97ZTNA/image2022-11-17%2013%3A55%3A44.png)

# 问题2

8090端口无法访问

检查ghana日志: /home/admin/logs/oms_console_std*.log

```
2022-11-17 12:59:01 ERROR o.s.b.SpringApplication - Application run failed
java.lang.IllegalArgumentException: spring.application.name must be configured!
        at org.springframework.util.Assert.isTrue(Assert.java:118)
        at com.alipay.sofa.tracer.boot.listener.SofaTracerConfigurationListener.onApplicationEvent(SofaTracerConfigurationListener.java:57)
        at com.alipay.sofa.tracer.boot.listener.SofaTracerConfigurationListener.onApplicationEvent(SofaTracerConfigurationListener.java:41)
        at org.springframework.context.event.SimpleApplicationEventMulticaster.doInvokeListener(SimpleApplicationEventMulticaster.java:172)
        at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:165)
        at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:139)
        at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:127)
        at org.springframework.boot.context.event.EventPublishingRunListener.environmentPrepared(EventPublishingRunListener.java:76)
        at org.springframework.boot.SpringApplicationRunListeners.environmentPrepared(SpringApplicationRunListeners.java:53)
        at org.springframework.boot.SpringApplication.prepareEnvironment(SpringApplication.java:342)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:305)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1215)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1204)
        at com.alipay.dss.endpoint.SOFABootRestApplication.main(SOFABootRestApplication.java:32)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:51)
        at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:52)
``` 

发现配置文件不存在: /home/ds/ghana/config/application-oms.properties

手工运行启动脚本中的python指令, 重新初始化配置文件: 

```
python -m omsflow.scripts.units.oms_cluster_config --check /home/admin/conf/config.yaml
// 正常
 
python -m omsflow.scripts.units.oms_init_manager --init-db
//报错, 连接元数据MySQL库报错
``` 

手工连接MySQL库, 发现密码插件问题: 

```
[root@ubuntu ~]# mysql -h10.186.62.73 -P8027 -utest -ptest
ERROR 2059 (HY000): Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib64/mysql/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory
``` 

修正密码插件配置后, python脚本仍然报错: 

```
[root@ubuntu ~]# python -m omsflow.scripts.units.oms_init_manager --init-db

/root/omsflow
use config /home/admin/conf/config.yaml

no drop task

no init task

query {'charset': 'utf8'} {'username': 'test', 'database': 'oms_cm', 'host': '10.186.62.73', 'password_original': 'test', 'drivername': 'mysql+pymysql', 'query': {'charset': 'utf8'}, 'port': 8027}
2022-11-17 15:52:24,146-Oms-DEBUG units.create_engine_wrapper.70 :create a new sqlalchemy engine wrapped Engine(mysql+pymysql://test:***@10.186.62.73:8027/oms_cm?charset=utf8)
Traceback (most recent call last):
  File "/usr/lib64/python2.7/runpy.py", line 162, in _run_module_as_main
    "__main__", fname, loader, pkg_name)
  File "/usr/lib64/python2.7/runpy.py", line 72, in _run_code
    exec code in run_globals
  File "/root/omsflow/scripts/units/oms_init_manager.py", line 485, in <module>
    main()
  File "/root/omsflow/scripts/units/oms_init_manager.py", line 473, in main
    init_manager.init_database()
  File "/root/omsflow/scripts/units/oms_init_manager.py", line 57, in init_database
    self.get_cm_engine()
  File "/root/omsflow/scripts/units/oms_init_manager.py", line 48, in get_cm_engine
    return self.get_safe_engine('_drc_cm_meta_uri')
  File "/root/omsflow/scripts/units/oms_init_manager.py", line 40, in get_safe_engine
    if not database_exists(engine.url):
  File "/usr/lib/python2.7/site-packages/sqlalchemy_utils/functions/database.py", line 479, in database_exists
    return bool(get_scalar_result(engine, text))
  File "/usr/lib/python2.7/site-packages/sqlalchemy_utils/functions/database.py", line 448, in get_scalar_result
    result_proxy = engine.execute(sql)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/base.py", line 2165, in execute
    connection = self._contextual_connect(close_with_result=True)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/base.py", line 2226, in _contextual_connect
    self._wrap_pool_connect(self.pool.connect, None),
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/base.py", line 2262, in _wrap_pool_connect
    return fn()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 354, in connect
    return _ConnectionFairy._checkout(self)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 751, in _checkout
    fairy = _ConnectionRecord.checkout(pool)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 483, in checkout
    rec = pool._do_get()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/impl.py", line 138, in _do_get
    self._dec_overflow()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/util/langhelpers.py", line 68, in __exit__
    compat.reraise(exc_type, exc_value, exc_tb)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/impl.py", line 135, in _do_get
    return self._create_connection()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 299, in _create_connection
    return _ConnectionRecord(self)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 428, in __init__
    self.__connect(first_connect_check=True)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 630, in __connect
    connection = pool._invoke_creator(self)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/strategies.py", line 114, in connect
    return dialect.connect(*cargs, **cparams)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/default.py", line 453, in connect
    return self.dbapi.connect(*cargs, **cparams)
  File "/usr/lib/python2.7/site-packages/pymysql/__init__.py", line 90, in Connect
    return Connection(*args, **kwargs)
  File "/usr/lib/python2.7/site-packages/pymysql/connections.py", line 706, in __init__
    self.connect()
  File "/usr/lib/python2.7/site-packages/pymysql/connections.py", line 931, in connect
    self._get_server_information()
  File "/usr/lib/python2.7/site-packages/pymysql/connections.py", line 1269, in _get_server_information
    self.server_charset = charset_by_id(lang).name
  File "/usr/lib/python2.7/site-packages/pymysql/charset.py", line 38, in by_id
    return self._by_id[id]
KeyError: 255
``` 

<https://stackoverflow.com/questions/45368336/error-keyerror-255-when-executing-pymysql-connect>

更换MySQL版本为 5.7 (之前使用8.0), 重启服务后可恢复正常

```
supervisorctl restart crond nginx oms_console oms_drc_cm oms_drc_supervisor sshd
supervisorctl status crond nginx oms_console oms_drc_cm oms_drc_supervisor sshd
``` 

# 问题3

![image2022-11-17 21:11:57.png](/assets/01KJBYV7TCGTDECT1W1G97ZTNA/image2022-11-17%2021%3A11%3A57.png)

在 ghana/Ghana/common-error.log 中找到相关日志: 

```
2022-11-17 21:07:44.283 [runningProjectHandleExecutor-3] ERROR c.a.o.s.s.ProjectHandler 58 - [021227c6-44f5-487c-99a1-e6ca911e33d6] Exception Stack:
com.alipay.oms.dto.exception.OmsTimeoutException: [TIMEOUT_EXCEPTION] {"operation":"PreCheckAction doRunningAction timeout.projectId p_48j1ugy38axs"}
        at com.alipay.oms.service.impl.action.step.PreCheckAction.doRunningAction(PreCheckAction.java:67)
        at com.alipay.oms.service.scheduler.AbstractProjectHandler.doActionForRunningSteps(AbstractProjectHandler.java:167)
        at com.alipay.oms.service.scheduler.AbstractProjectHandler.innerHandleRunningProject(AbstractProjectHandler.java:93)
        at com.alipay.oms.service.scheduler.ProjectHandler.handleRunningProject(ProjectHandler.java:38)
        at com.alipay.oms.service.scheduler.ProjectHandler$$FastClassBySpringCGLIB$$1947796d.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:750)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.interceptor.AsyncExecutionInterceptor.lambda$invoke$0(AsyncExecutionInterceptor.java:115)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:853)
``` 

源码位置: 在 /u01/ds/ghana/boot/Ghana-endpoint-1.0.0-executable.jar 中, 找到 Ghana-biz-shared-1.0.0.jar

在 ghana/Ghana/oms-scheduler.log 中也找到相关日志: 

```
2022-11-17 21:05:13.400 [INFO] [runningProjectHandleExecutor-1] [110d75ba-b6cd-4795-b565-d6ea75dccb15] [MigrationProjectScan_48j1ulgjmk9s] [p_48j1ugy38axs] [PRE_CHECK:s_48j1ugz76ogw] Begin doActionForRunningSteps.
2022-11-17 21:05:13.404 [ERROR] [runningProjectHandleExecutor-1] [110d75ba-b6cd-4795-b565-d6ea75dccb15] Exception Stack:
com.alipay.oms.dto.exception.OmsTimeoutException: [TIMEOUT_EXCEPTION] {"operation":"PreCheckAction doRunningAction timeout.projectId p_48j1ugy38axs"}
        at com.alipay.oms.service.impl.action.step.PreCheckAction.doRunningAction(PreCheckAction.java:67)
        at com.alipay.oms.service.scheduler.AbstractProjectHandler.doActionForRunningSteps(AbstractProjectHandler.java:167)
        at com.alipay.oms.service.scheduler.AbstractProjectHandler.innerHandleRunningProject(AbstractProjectHandler.java:93)
        at com.alipay.oms.service.scheduler.ProjectHandler.handleRunningProject(ProjectHandler.java:38)
        at com.alipay.oms.service.scheduler.ProjectHandler$$FastClassBySpringCGLIB$$1947796d.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:750)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.interceptor.AsyncExecutionInterceptor.lambda$invoke$0(AsyncExecutionInterceptor.java:115)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:853)
...
``` 

从代码逻辑, 知道 其比较的是OMS任务的Modified:

![image2022-11-17 21:39:39.png](/assets/01KJBYV7TCGTDECT1W1G97ZTNA/image2022-11-17%2021%3A39%3A39.png)

检查元数据库中的数据:

![image2022-11-17 21:39:59.png](/assets/01KJBYV7TCGTDECT1W1G97ZTNA/image2022-11-17%2021%3A39%3A59.png)

可能是由于时区问题, 导致时间比较出错. 

修正服务器时区, 报错消失

# 问题4

数据迁移任务, 会卡在预检查环节: 

![image2022-11-18 11:30:32.png](/assets/01KJBYV7TCGTDECT1W1G97ZTNA/image2022-11-18%2011%3A30%3A32.png)

相关日志: oms-scheduler.log.2022-11-17

![image2022-11-18 19:8:19.png](/assets/01KJBYV7TCGTDECT1W1G97ZTNA/image2022-11-18%2019%3A8%3A19.png)

Tips:

  - ./oms_console_stdout.log 能找到OMS 下到元数据库中的SQL

结果: 放置一个周末后, 任务自动失败. 未找到原因

# 复制链路

在OMS UI上, 建立一个数据迁移链路. 

在OMS进程中, 会多出一个java进程: 

```
ds        4085  6.3  1.5 9108840 247568 pts/0  Sl   15:11   0:30 java -server -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/u01/ds/bin/../run/10.186.62.73-9000:p_48wfv693y1b4_dest-000-0:0000000001 -Djava.library.path=/u01/ds/bin/../plugins/jdbc_connector/ -Duser.home=/home/ds -Dlogging.path=/u01/ds/bin/../run/10.186.62.73-9000:p_48wfv693y1b4_dest-000-0:0000000001/logs -cp jdbc_connector.jar:jdbc-source-store.jar:jdbc-sink-ob-mysql.jar com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap -c conf -s com.oceanbase.oms.connector.jdbc.source.store.StoreSource -d com.oceanbase.oms.connector.jdbc.sink.obmysql.OBMySQLJdbcSink -t 10.186.62.73-9000:p_48wfv693y1b4_dest-000-0:0000000001 start
``` 

通过fd找到运行目录: 

![image2022-11-21 15:22:56.png](/assets/01KJBYV7TCGTDECT1W1G97ZTNA/image2022-11-21%2015%3A22%3A56.png)

与p_48wfv693y1b4相关的进程有以下几个: 

```
[root@ubuntu ~]# ps aux | grep p_48wfv693y1b4
ds        1843  0.0  0.0 367240 14740 pts/0    Sl   15:10   0:02 /u01/ds/store/store7100/bin/metadata_builder -b p_48wfv693y1b4_source-000-0.0000000001 -h 10.186.62.73 -P 5731 -u root -p msandbox -e UTF8 -c ./metadata/conf/my.cnf -v 0 -m ./metadata/meta -z 10240 -o 1669014607 -x false -j 12 -k 504 -l ./meta.log -s /home/ds/mysql57 -t 10 -R 2003,2004,2006,2013 -f false -T '*' -W '*|{filter.conditions}|test.a' -B '{filter.blacklist}|test.sp:[CreateTable]' -M . -n 0

ds        2038  0.0  0.0  13304  2780 pts/0    S    15:10   0:00 sh -c /home/ds/mysql57/bin/mysqld_safe --defaults-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/conf/my.cnf --basedir=/home/ds/mysql57 --datadir=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/data  --tmpdir=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/tmp  --pid-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/run/mysqld.pid  --socket=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/run/mysql.sock  --log-error=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/alert.log  --general_log_file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/general.log  --slow_query_log_file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/slow.log --plugin-dir=/home/ds/mysql57/lib/plugin --open-files-limit=65535 1>/dev/null 2>&1

ds        2039  0.0  0.0  13440  3108 pts/0    S    15:10   0:00 /bin/sh /home/ds/mysql57/bin/mysqld_safe --defaults-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/conf/my.cnf --basedir=/home/ds/mysql57 --datadir=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/data --tmpdir=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/tmp --pid-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/run/mysqld.pid --socket=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/run/mysql.sock --log-error=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/alert.log --general_log_file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/general.log --slow_query_log_file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/slow.log --plugin-dir=/home/ds/mysql57/lib/plugin --open-files-limit=65535

ds        3142  0.1  0.4 2348164 69280 pts/0   Sl   15:10   0:46 /home/ds/mysql57/bin/mysqld --defaults-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/conf/my.cnf --basedir=/home/ds/mysql57 --datadir=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/data --plugin-dir=/home/ds/mysql57/lib/plugin --tmpdir=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/tmp --general-log-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/general.log --slow-query-log-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/slow.log --log-error=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/log/alert.log --open-files-limit=65535 --pid-file=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/run/mysqld.pid --socket=/u01/ds/store/store7100/p_48wfv693y1b4_source-000-0.0000000001/metadata/run/mysql.sock

ds        4085  4.7  1.8 9643400 309140 pts/0  Sl   15:11  24:46 java -server -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/u01/ds/bin/../run/10.186.62.73-9000:p_48wfv693y1b4_dest-000-0:0000000001 -Djava.library.path=/u01/ds/bin/../plugins/jdbc_connector/ -Duser.home=/home/ds -Dlogging.path=/u01/ds/bin/../run/10.186.62.73-9000:p_48wfv693y1b4_dest-000-0:0000000001/logs -cp jdbc_connector.jar:jdbc-source-store.jar:jdbc-sink-ob-mysql.jar com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap -c conf -s com.oceanbase.oms.connector.jdbc.source.store.StoreSource -d com.oceanbase.oms.connector.jdbc.sink.obmysql.OBMySQLJdbcSink -t 10.186.62.73-9000:p_48wfv693y1b4_dest-000-0:0000000001 start
``` 

  - 问题1: 密码明文写在的进程命令中
  - 启动了一个MySQLd实例, 用途不明  
  

下一步: 使用IntelliJ, 导入相关Jar, 进行源码分析
