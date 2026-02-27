---
title: 20231027 - 对OMS的日志系统分析
confluence_page_id: 2589295
created_at: 2023-10-27T07:42:37+00:00
updated_at: 2023-11-08T05:05:55+00:00
---

# 目录

# 端口分析

UI端口 8089 

nginx配置: /home/ds/ghana/config/tengine.conf, 将8089请求转发到 8090

8090 由Ghana进程处理

# 日志分析方法

使用arthas命令: 

```
stack org.slf4j.Logger info
``` 

可以打印出输出日志的相关堆栈

# 进程 org.apache.kafka.connect.cli.ConnectDRCDeliver

```
ls -alh /proc/67578/fd | grep -v "pipe:" | grep -v "socket:" | grep -v "/dev/random" | grep -v "/dev/urandom" | grep -v "/dev/null" | grep -v "anon_inode" | egrep -v '\.jar$' | less
``` 

结果: 

```
dr-x------ 2 ds ds  0 Oct 30 17:56 .
dr-xr-xr-x 9 ds ds  0 Oct 30 17:56 ..
l-wx------ 1 ds ds 64 Oct 30 17:57 1 -> /u01/ds/store/store7101/log/store.log
l-wx------ 1 ds ds 64 Oct 30 17:57 118 -> /u01/ds/store/store7101/log/connector/connector.log
l-wx------ 1 ds ds 64 Oct 30 17:57 119 -> /u01/ds/store/store7101/log/connector/metric/fetcher.log
l-wx------ 1 ds ds 64 Oct 30 17:57 120 -> /u01/ds/store/store7101/log/connector/metric/analyzer.log
l-wx------ 1 ds ds 64 Oct 30 17:57 121 -> /u01/ds/store/store7101/log/connector/metric/aggregator.log
l-wx------ 1 ds ds 64 Oct 30 17:57 122 -> /u01/ds/store/store7101/log/connector/metric/converter.log
l-wx------ 1 ds ds 64 Oct 30 17:57 123 -> /u01/ds/store/store7101/log/connector/metric/selector.log
l-wx------ 1 ds ds 64 Oct 30 17:57 124 -> /u01/ds/store/store7101/log/connector/metric/metric.log
l-wx------ 1 ds ds 64 Oct 30 17:57 125 -> /u01/ds/store/store7101/log/connector/config.log
l-wx------ 1 ds ds 64 Oct 30 17:56 2 -> /u01/ds/store/store7101/log/store.log
l-wx------ 1 ds ds 64 Oct 30 17:57 3 -> /u01/ds/store/store7101/log/congo_0
l-wx------ 1 ds ds 64 Oct 30 17:57 4 -> /u01/ds/store/store7101/queue_log_0
lrwx------ 1 ds ds 64 Oct 30 17:57 7 -> /u01/ds/store/store7101/DLlib/complite.log
l-wx------ 1 ds ds 64 Oct 30 17:57 8 -> /u01/ds/store/store7101/ORACLE_np_56wquq78g37k_56wr24sdtun4-1-0.0001000001/-.0/queue_log_0
``` 

日志配置文件: 

  - /u01/ds/store/store7100/kafka/config/logback.xml  

```
<?xml version="1.0"?>
<configuration>
    <property name="ROOT" value="${log.file.root:-./log/connector}" />

    <property name="METRIC-PATH" value="${ROOT}/metric" />
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%p] [%t] [%m]\(%F:%L\)%n</pattern>
        </encoder>
    </appender>

    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${ROOT}/connector.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${ROOT}/connector.%d{yyyy-MM-dd}.log.gz</FileNamePattern>
            <maxHistory>14</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%p] [%t] [%m]\(%F:%L\)%n
            </pattern>
        </layout>
    </appender>

    <appender name="FETCHER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${METRIC-PATH}/fetcher.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${METRIC-PATH}/fetcher.%d{yyyy-MM-dd_HH}_%i.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
            <maxFileSize>4096MB</maxFileSize>
            <totalSizeCap>49152MB</totalSizeCap>
        </rollingPolicy>
        <immediateFlush>false</immediateFlush>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%m%n
            </pattern>
        </layout>
    </appender>

    <appender name="ANALYZER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${METRIC-PATH}/analyzer.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${METRIC-PATH}/analyzer.%d{yyyy-MM-dd_HH}_%i.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
            <maxFileSize>2048MB</maxFileSize>
            <totalSizeCap>8192MB</totalSizeCap>
        </rollingPolicy>
        <immediateFlush>false</immediateFlush>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%m%n
            </pattern>
        </layout>
    </appender>

    <appender name="AGGREGATOR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${METRIC-PATH}/aggregator.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${METRIC-PATH}/aggregator.%d{yyyy-MM-dd_HH}_%i.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
            <maxFileSize>2048MB</maxFileSize>
            <totalSizeCap>8192MB</totalSizeCap>
        </rollingPolicy>
        <immediateFlush>false</immediateFlush>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%m%n
            </pattern>
        </layout>
    </appender>

    <appender name="CONVERTER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${METRIC-PATH}/converter.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${METRIC-PATH}/converter.%d{yyyy-MM-dd_HH}_%i.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
            <maxFileSize>2048MB</maxFileSize>
            <totalSizeCap>8192MB</totalSizeCap>
        </rollingPolicy>
        <immediateFlush>false</immediateFlush>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%m%n
            </pattern>
        </layout>
    </appender>

    <appender name="SELECTOR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${METRIC-PATH}/selector.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${METRIC-PATH}/selector.%d{yyyy-MM-dd_HH}_%i.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
            <maxFileSize>2048MB</maxFileSize>
            <totalSizeCap>8192MB</totalSizeCap>
        </rollingPolicy>
        <immediateFlush>false</immediateFlush>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%m%n
            </pattern>
        </layout>
    </appender>

    <appender name="METRICS" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${METRIC-PATH}/metric.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${METRIC-PATH}/metric.%d{yyyy-MM-dd}.log.gz</FileNamePattern>
            <maxHistory>14</maxHistory>
        </rollingPolicy>
        <immediateFlush>false</immediateFlush>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] %m%n
            </pattern>
        </layout>
    </appender>

    <appender name="CONFIG" class="ch.qos.logback.core.FileAppender">
        <File>${ROOT}/config.log</File>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] %m%n
            </pattern>
        </layout>
    </appender>

    <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator">
        <resetJUL>true</resetJUL>
    </contextListener>
    <logger name="stdout" level="info" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="fetcher" level="info" additivity="false">
        <appender-ref ref="FETCHER"/>
    </logger>
    <logger name="analyzer" level="info" additivity="false">
        <appender-ref ref="ANALYZER" />
    </logger>
    <logger name="aggregator" level="info" additivity="false">
        <appender-ref ref="AGGREGATOR" />
    </logger>
    <logger name="converter" level="info" additivity="false">
        <appender-ref ref="CONVERTER" />
    </logger>
    <logger name="selector" level="info" additivity="false">
        <appender-ref ref="SELECTOR" />
    </logger>
    <logger name="metrics" level="info" additivity="false">
        <appender-ref ref="METRICS" />
    </logger>
    <logger name="config" level="info" additivity="false">
        <appender-ref ref="CONFIG" />
    </logger>
    <logger name="org.I0Itec.zkclient" level="ERROR"/>
    <logger name="org.apache.zookeeper" level="ERROR"/>
    <root>
        <level value="info" />
        <appender-ref ref="file" />
    </root>
</configuration>[root@R740-26 ~]#
```

## 日志配置

命令行参数: 

```
-Dkafka.logs.dir=/home/ds/store/store7100/kafka/bin/../logs 
-Dlogback.configurationFile=file:./kafka/bin/../config/logback.xml 
-Dlog4j.configuration=file:./kafka/bin/../config/connect-log4j.properties 
``` 

log4j配置: 

```
log4j.rootLogger=INFO, FILE

log4j.appender.FILE=org.apache.log4j.DailyRollingFileAppender
log4j.appender.FILE.ImmediateFlush=true
log4j.appender.FILE.Append=true
log4j.appender.FILE.File=log/connector/connector.log
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
log4j.appender.FILE.layout.ConversionPattern=[%d] %p [%t] %m (%c:%L)%n
log4j.appender.FILE.MaxFileSize=10MB
log4j.appender.FILE.MaxBackupIndex=10

log4j.logger.org.apache.zookeeper=ERROR
log4j.logger.org.I0Itec.zkclient=ERROR
``` 

## log/connector/connector.log

  - /u01/ds/store/store7100/kafka/config/logback.xml 定义的root logger
  - 在log4j配置中, 也是root logger

## log/connector/metric/metric.log

写日志的代码: com.taobao.drc.logminer.metric.LogminerMetric#run, 由该线程遍历所有实现了 com.taobao.drc.logminer.metric.MetricElement 的类, 遍历的实际方法委托给 com.taobao.drc.logminer.metric.visitor.impl.PerfVisitor, 其中包含各类的性能指标采集方式, 举例: 

![image2023-10-27 19:53:20.png](/assets/01KJBZ3FQT8FKX47BF6Q8C4X7J/image2023-10-27%2019%3A53%3A20.png)

其中包含性能指标的类是: 

```
LogRecordConverterCore
LogRecordConverterPrepare
OracleBackRecordQueue
OracleLogExtractor
OracleLogKVStoreAggregator
LogEntryFetcher
LogMinerConnectorTask.TaskRecordQueue
``` 

## log/connector/config.log

定义在: com.taobao.drc.logminer.logger.Log, 仅在配置初始化时打印日志 (com.taobao.drc.logminer.LogminerWrapper#init)

配置初始化是 com.taobao.drc.logminer.connect.LogMinerConnectorTask#start 时进行, (kafka开启任务时)

kafka的启动配置中, 使用配置文件 ./conf/deliver2store.conf, 启动类是com.taobao.drc.logminer.connect.LogMinerConnector, 其中指定了LogMinerConnectorTask作为处理任务

## log/connector/metric/, fetcher, analyzer, aggregator, converter, selector

都定义在: com.taobao.drc.logminer.logger.Log

但未见使用

## 其他日志

使用arthas诊断, logger列表中未见到congo_0. 也就是说congo_0不是作为日志文件导入. 

该进程的父进程是store, 其他日志都是从父进程继承过来, 但没有正确关闭. 

# 进程 CM (jetty)

进程链:

```
/usr/bin/python2 /usr/bin/supervisord -c /etc/supervisor/supervisord.conf --nodaemon
	- bash /home/admin/conf/command/start_oms_cm.sh
		- java -cp /home/ds/cm/package/deployapp/lib/commons-daemon.jar:/home/ds/cm/package/jetty/start.jar -server -Xmx4g -Xms4g -Xmn3g -Dorg.eclipse.jetty.util.URI.charset=utf-8 -Dorg.eclipse.jetty.server.Request.maxFormContentSize=0 -Dorg.eclipse.jetty.server.Request.maxFormKeys=20000 -DSTOP.PORT=8089 -DSTOP.KEY=cm -Djetty.base=/home/ds/cm/package/deployapp org.eclipse.jetty.start.Main
``` 

jetty的启动war: /home/ds/cm/package/deployapp/webapps/cm.war

servlet配置文件, 在cm.war中: WEB-INF/web.xml

  - /metrics/*, 由com.codahale.metrics.servlets.AdminServlet处理
  - 其他路由, 由/WEB-INF/spring/servlet-context.xml定义
    - 组件扫描路径: com.alibaba.drc.controller / com.alibaba.drc.api.action / com.alibaba.drc.entity.guid.action

日志文件列表: 

```
l-wx------ 1 root root 64 Oct 30 21:47 258 -> /home/admin/logs/cm/log/dao.log
l-wx------ 1 root root 64 Oct 30 21:47 259 -> /home/admin/logs/cm/log/dao-digest.log
l-wx------ 1 root root 64 Oct 30 21:47 260 -> /home/admin/logs/cm/log/cache-digest.log
l-wx------ 1 root root 64 Oct 30 21:47 261 -> /home/admin/logs/cm/log/lock-service.log
l-wx------ 1 root root 64 Oct 30 21:47 262 -> /home/admin/logs/cm/log/idb-opsdba.log
l-wx------ 1 root root 64 Oct 30 21:47 263 -> /home/admin/logs/cm/log/qconsole-digest.log
l-wx------ 1 root root 64 Oct 30 21:47 264 -> /home/admin/logs/cm/log/cache-service.log
l-wx------ 1 root root 64 Oct 30 21:47 265 -> /home/admin/logs/cm/log/cm-web.log
l-wx------ 1 root root 64 Oct 30 21:47 266 -> /home/admin/logs/cm/log/common-default.log
l-wx------ 1 root root 64 Oct 30 21:47 267 -> /home/admin/logs/cm/log/common-error.log
l-wx------ 1 root root 64 Oct 30 21:47 268 -> /home/admin/logs/cm/log/service.log
l-wx------ 1 root root 64 Oct 30 21:47 269 -> /home/admin/logs/cm/log/job.log
l-wx------ 1 root root 64 Oct 30 21:47 270 -> /home/admin/logs/cm/log/commit.log
l-wx------ 1 root root 64 Oct 30 21:47 271 -> /home/admin/logs/cm/log/checkPoint.log
l-wx------ 1 root root 64 Oct 30 21:47 272 -> /home/admin/logs/cm/log/monitor.log
l-wx------ 1 root root 64 Oct 30 21:47 273 -> /home/admin/logs/cm/log/upgrade.log
l-wx------ 1 root root 64 Oct 30 21:47 33 -> /home/admin/logs/cm/log/20231030.cm-api.log
lr-x------ 1 root root 64 Oct 30 21:47 34 -> /u01/ds/cm/package/deployapp/webapps/cm.war
``` 

日志配置文件: cm.war:WEB-INF/classes/log4j2.xml: 

  - 重要的APPENDER: 
    - DEFAULT-APPENDER 和 ERROR-APPENDER: 都只包括WARN及以上级别日志, 配置相同 (疑似bug). 文件名: common-default.log/common-error.log
    - commonlog: INFO及以上级别日志. 文件名: cm-web.log
  - Logger配置: 

```
    <Loggers>
        <!-- 3rdparty Loggers -->
        <Logger name="org.springframework" additivity="false">
            <AppenderRef ref="commonlog" />
        </Logger>
        <Logger name="org.apache.commons" additivity="false">
            <AppenderRef ref="commonlog" />
        </Logger>
        <Logger name="com.alibaba.druid" additivity="false">
            <AppenderRef ref="commonlog" />
        </Logger>
        <Logger name="org.apache.velocity" additivity="false"> 
            <AppenderRef ref="commonlog" />
        </Logger>
        <Logger name="org.apache.http" additivity="false"> 
            <AppenderRef ref="commonlog" />
        </Logger>
        <Logger name="com.alibaba.drc.client.commit" additivity="false">
            <AppenderRef ref="commitLog"/>
        </Logger>
        <Logger name="com.alibaba.drc.service.impl.OffsetServiceImpl" additivity="false">
            <AppenderRef ref="checkPointLog"/>
        </Logger>

        <!-- Application Loggers -->
        <Logger name="com.alibaba.drc.common.utils.monitor" additivity="false">
            <AppenderRef ref="monitorlog" />
        </Logger>
        <Logger name="com.alibaba.drc.service.upgrade" additivity="false">
            <AppenderRef ref="upgradelog" />
        </Logger>
        <Logger name="com.alibaba.drc" additivity="false">
            <AppenderRef ref="commonlog" />
        </Logger>
        <Logger name="org.quartz" additivity="false">
            <AppenderRef ref="joblog" />
        </Logger>
        <Logger name="com.alibaba.drc.service" additivity="false">
            <AppenderRef ref="servicelog" />
        </Logger>
        <Logger name="org.mybatis" additivity="false">
            <AppenderRef ref="daolog" />
        </Logger>
        <Logger name="org.apache.ibatis" additivity="false">
            <AppenderRef ref="daolog" />
        </Logger>
        <Logger name="com.alibaba.drc.dao" additivity="false">
            <AppenderRef ref="daolog"></AppenderRef>
        </Logger>
        <Logger name="DAL-DIGEST" additivity="false">
            <AppenderRef ref="dalDigest" />
        </Logger>
        <Logger name="CACHE-DIGEST" additivity="false">
            <AppenderRef ref="cacheDigest" />
        </Logger>
        <Logger name="CACHE-SERVICE" additivity="false">
            <AppenderRef ref="cacheService" />
        </Logger>

        <Logger name="LOCK-SERVICE" additivity="false">
            <AppenderRef ref="lockService" />
        </Logger>

        <Logger name="IDB-OPSDBA" additivity="false">
            <AppenderRef ref="idbOpsdba" />
        </Logger>

        <Logger name="QCONSOLE-DIGEST" additivity="false">
            <AppenderRef ref="qconsole-digest" />
        </Logger>

        <Root level="INFO">
            <AppenderRef ref="ERROR-APPENDER" />
            <AppenderRef ref="DEFAULT-APPENDER" />
        </Root>
```

日志格式中, 有统一的traceId: 

```
<Property name="patternLayout">%d{yyyy-MM-dd HH:mm:ss} %p %l - [%X{traceId}] %m%n</Property>
``` 

改善建议: 在UI上, 报错标记好traceID, 后端日志按照trace ID + 时间戳 进行排序, 方便查看

# 进程 Ghana

日志列表: 

```
l-wx------ 1 root root 64 Oct 30 22:26 10 -> /home/admin/logs/ghana/Ghana/oms-integration.log
l-wx------ 1 root root 64 Oct 30 22:26 11 -> /home/admin/logs/ghana/Ghana/catalina.log
l-wx------ 1 root root 64 Oct 30 22:26 12 -> /home/admin/logs/ghana/Ghana/dbcat.log
l-wx------ 1 root root 64 Oct 30 22:26 13 -> /home/admin/logs/ghana/Ghana/xflush.log
l-wx------ 1 root root 64 Oct 30 22:26 14 -> /home/admin/logs/ghana/Ghana/dss-operation.log
l-wx------ 1 root root 64 Oct 30 22:26 15 -> /home/admin/logs/ghana/Ghana/meta-db.log
l-wx------ 1 root root 64 Oct 30 22:26 16 -> /home/admin/logs/ghana/Ghana/security-info.log
l-wx------ 1 root root 64 Oct 30 22:26 17 -> /home/admin/logs/ghana/Ghana/oms-scheduler.log
l-wx------ 1 root root 64 Oct 30 22:26 18 -> /home/admin/logs/ghana/Ghana/oms-ha.log
l-wx------ 1 root root 64 Oct 30 22:26 19 -> /home/admin/logs/ghana/Ghana/oms-step.log
l-wx------ 1 root root 64 Oct 30 22:26 20 -> /home/admin/logs/ghana/Ghana/database.log
l-wx------ 1 root root 64 Oct 30 22:26 21 -> /home/admin/logs/ghana/Ghana/forward-supervisor.log
l-wx------ 1 root root 64 Oct 30 22:26 22 -> /home/admin/logs/ghana/Ghana/oms-api.log
l-wx------ 1 root root 64 Oct 30 22:26 23 -> /home/admin/logs/ghana/Ghana/connection-error.log
l-wx------ 1 root root 64 Oct 30 22:26 24 -> /home/admin/logs/ghana/Ghana/omc.log
l-wx------ 1 root root 64 Oct 30 22:26 25 -> /home/admin/logs/ghana/Ghana/oms-web.log
l-wx------ 1 root root 64 Oct 30 22:26 26 -> /home/admin/logs/ghana/Ghana/oms-cmsdk.log
l-wx------ 1 root root 64 Oct 30 22:26 27 -> /home/admin/logs/ghana/Ghana/event-tracking/project-execute-failed.log
l-wx------ 1 root root 64 Oct 30 22:26 28 -> /home/admin/logs/ghana/Ghana/event-tracking/step-execute-time-cost.log
l-wx------ 1 root root 64 Oct 30 22:26 29 -> /home/admin/logs/ghana/Ghana/event-tracking/project_status.log
l-wx------ 1 root root 64 Oct 30 22:26 3 -> /home/admin/logs/ghana/gc.log.0.current
l-wx------ 1 root root 64 Oct 30 22:26 30 -> /home/admin/logs/ghana/Ghana/event-tracking/incr-sync-step-delay.log
l-wx------ 1 root root 64 Oct 30 22:26 31 -> /home/admin/logs/ghana/Ghana/event-tracking/api-call.log
l-wx------ 1 root root 64 Oct 30 22:26 32 -> /home/admin/logs/ghana/Ghana/event-tracking/error-code-feedback.log
l-wx------ 1 root root 64 Oct 30 22:26 33 -> /home/admin/logs/ghana/Ghana/event-tracking/tracking_exception.log
l-wx------ 1 root root 64 Oct 30 22:26 34 -> /home/admin/logs/ghana/Ghana/oms-alarm.log
l-wx------ 1 root root 64 Oct 30 22:26 35 -> /home/admin/logs/ghana/Ghana/upgrade_4.2.0/out.log
l-wx------ 1 root root 64 Oct 30 22:26 36 -> /home/admin/logs/ghana/Ghana/event-tracking/oms-api.log
l-wx------ 1 root root 64 Oct 30 22:26 37 -> /home/admin/logs/ghana/Ghana/event-tracking/k8s-event.log
l-wx------ 1 root root 64 Oct 30 22:26 38 -> /home/admin/logs/ghana/Ghana/event-tracking/alarm-schedule.log
l-wx------ 1 root root 64 Oct 30 22:26 39 -> /home/admin/logs/ghana/Ghana/health-check.log
l-wx------ 1 root root 64 Oct 30 22:26 8 -> /home/admin/logs/ghana/Ghana/common-error.log
l-wx------ 1 root root 64 Oct 30 22:26 9 -> /home/admin/logs/ghana/Ghana/common-default.log
l-wx------ 1 root root 64 Oct 30 22:26 92 -> /root/logs/mcms/loggerMonitorKeyCollect.log
``` 

日志配置: classpath:logback-spring.xml: 

  - 重要的APPENDER: 
    - ROOT-APPENDER 包括 ${logging.level} 及以上级别日志
    - ERROR-APPENDER 包括 error及以上级别日志
    - 有一些appender, 日志格式中没有traceID
  - logger配置: 

```
  <logger name="com.alipay.dss.core.util.HttpUtils" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-INTEGRATION-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>
    <logger name="com.alipay.oms.common.service.integration.cm.CmClientUtil" level="${logging.level}"
            additivity="false">
        <appender-ref ref="OMS-INTEGRATION-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>
    <logger name="com.alipay.oms.common.interceptor.DataAccessInterceptor" level="${logging.level}" additivity="false">
        <appender-ref ref="DAL-APPENDER"/>
    </logger>

    <logger name="com.alipay.common.security" level="${logging.level}" additivity="false">
        <appender-ref ref="ALIPAY-COMMON-SECURITY-APPENDER"/>
    </logger>

    <logger name="com.alipay.dss.core.operation" level="${logging.level}" additivity="false">
        <appender-ref ref="OPERATION-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="org.apache.tomcat" level="${logging.level}" additivity="false">
        <appender-ref ref="CATALINA-APPENDER"/>
    </logger>
    <logger name="org.apache.catalina" level="${logging.level}" additivity="false">
        <appender-ref ref="CATALINA-APPENDER"/>
    </logger>

    <logger name="com.oceanbase.obtools" level="${logging.level}" additivity="false">
        <appender-ref ref="DBCAT-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.scheduler.AbnormalWorkerHandler" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-HA-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.scheduler.OmsWorkerRegulator" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-HA-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.scheduler" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-SCHEDULER-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.impl.action.step" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-STEP-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.impl.ApiCommonService" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-API-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.vendor.aliyun.AliyunCommonService" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-API-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.common.aop.WebLogAspect" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-WEB-ASPECT-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.util.OmsDbUtil" level="${logging.level}" additivity="false">
        <appender-ref ref="DB-UTIL-APPENDER"/>
    </logger>

    <logger name="forward_supervisor" level="${logging.level}" additivity="false">
        <appender-ref ref="FORWARD_SUPERVISOR_APPENDER" />
    </logger>

    <logger name="com.alibaba.druid.pool" level="${logging.level}" additivity="false">
        <appender-ref ref="CONNECTION-ERROR-APPENDER"/>
    </logger>

    <logger name="org.springframework.jdbc.CannotGetJdbcConnectionException" level="${logging.level}" additivity="false">
        <appender-ref ref="CONNECTION-ERROR-APPENDER"/>
    </logger>

    <logger name="com.alipay.rds.scheduler" level="${logging.level}" additivity="false">
        <appender-ref ref="OMC-APPENDER"/>
    </logger>

    <logger name="com.oceanbase.oms.cm" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-CMSDK-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.scheduler.OmsAlarmJob" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-ALARM-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.facade.oms.alarm.AbstractOmsAlarmService" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-ALARM-APPENDER"/>
    </logger>

    <logger name="com.alipay.oms.service.impl.oms.alarm" level="${logging.level}" additivity="false">
        <appender-ref ref="OMS-ALARM-APPENDER"/>
    </logger>

    <logger name="projectExecuteFailedEventTrackingLogger" level="TRACE" additivity="false">
        <appender-ref ref="projectExecuteFailedEventTrackingAppender" />
    </logger>

    <logger name="stepExecuteTimeCostEventTrackingLogger" level="TRACE" additivity="false">
        <appender-ref ref="stepExecuteTimeCostEventTrackingAppender" />
    </logger>

    <logger name="projectStatusEventTrackingLogger" level="TRACE" additivity="false">
        <appender-ref ref="projectStatusEventTrackingAppender" />
    </logger>

    <logger name="incrSyncStepDelayEventTrackingLogger" level="TRACE" additivity="false">
        <appender-ref ref="incrSyncStepDelayEventTrackingAppender" />
    </logger>

    <logger name="apiCallEventTrackingLogger" level="TRACE" additivity="false">
        <appender-ref ref="apiCallEventTrackingAppender"/>
    </logger>

    <logger name="errorCodeFeedbackEventTrackingLogger" level="TRACE" additivity="false">
        <appender-ref ref="errorCodeFeedbackEventTrackingAppender" />
    </logger>

    <logger name="trackingExceptionLogger" level="TRACE" additivity="false">
        <appender-ref ref="trackingExceptionAppender" />
    </logger>

    <logger name="upgradeLogger" level="TRACE" additivity="false">
        <appender-ref ref="UPGRADE-LOG-APPENDER"/>
        <appender-ref ref="STDOUT"/>
    </logger>

    <logger name="omsApiLogger" level="TRACE" additivity="false">
        <appender-ref ref="OMS-API-LOG-APPENDER"/>
    </logger>

    <logger name="k8sEventLogger" level="TRACE" additivity="false">
        <appender-ref ref="K8S-EVENT-LOG-APPENDER"/>
    </logger>

    <logger name="alarmScheduleLogger" level="TRACE" additivity="false">
        <appender-ref ref="ALARM-SCHEDULE-LOG-APPENDER"/>
    </logger>

    <logger name="healthCheckLogger" level="${logging.level}" additivity="false">
        <appender-ref ref="HEALTH-CHECK-APPENDER" />
    </logger>

    <root level="${logging.level}">
        <appender-ref ref="ROOT-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </root>
```

样例分析: 

界面调用, 在oms-web.log可以找到记录 (注意trace ID)

```
oms-web.log:2023-10-31 00:15:27.925 [http-nio-8090-exec-2] INFO  c.a.o.c.a.WebLogAspect 103 - [a31f65fd-9127-4dd4-b147-adc24a71198b] URL = http://localhost:8090/v3/api
oms-web.log:2023-10-31 00:15:27.926 [http-nio-8090-exec-2] INFO  c.a.o.c.a.WebLogAspect 104 - [a31f65fd-9127-4dd4-b147-adc24a71198b] HTTP_METHOD = POST
oms-web.log:2023-10-31 00:15:27.927 [http-nio-8090-exec-2] INFO  c.a.o.c.a.WebLogAspect 105 - [a31f65fd-9127-4dd4-b147-adc24a71198b] IP = 172.16.169.1
oms-web.log:2023-10-31 00:15:27.927 [http-nio-8090-exec-2] INFO  c.a.o.c.a.WebLogAspect 110 - [a31f65fd-9127-4dd4-b147-adc24a71198b] METHOD = com.alipay.oms.v3.controller.TransferLinkController.getProject
oms-web.log:2023-10-31 00:15:27.928 [http-nio-8090-exec-2] INFO  c.a.o.c.a.WebLogAspect 113 - [a31f65fd-9127-4dd4-b147-adc24a71198b] ARGS = [{"acceptLanguage":"ZH","id":"np_56el1g1jvrs0","uid":"admin","useOss":false}]
oms-web.log:2023-10-31 00:15:28.218 [http-nio-8090-exec-2] INFO  c.a.o.c.a.WebLogAspect 159 - [a31f65fd-9127-4dd4-b147-adc24a71198b] RESPONSE : OmsApiReturnResult(success=true, errorDetail=null, code=null, message=null, advice=null, requestId=a31f65fd-9127-4dd4-b147-adc24a71198b, pageN
......
``` 

按trace ID, 在oms-integration.log中, 可以看到调用CM的API记录 (但没有返回值): 

```
oms-integration.log:2023-10-31 00:15:28.156 [INFO] - [a31f65fd-9127-4dd4-b147-adc24a71198b] [GET] [http://10.186.16.126:8088] [/] [200] [2] []
oms-integration.log:2023-10-31 00:15:28.161 [INFO] - [a31f65fd-9127-4dd4-b147-adc24a71198b] [GET] [http://10.186.16.126:8088] [/subtopic/detail] [200] [5] []
oms-integration.log:2023-10-31 00:15:28.169 [INFO] - [a31f65fd-9127-4dd4-b147-adc24a71198b] [GET] [http://10.186.16.126:8088] [/crawler/list] [200] [5] []
oms-integration.log:2023-10-31 00:15:28.177 [INFO] - [a31f65fd-9127-4dd4-b147-adc24a71198b] [GET] [http://10.186.16.126:8088] [/monitor/crawler/overview] [200] [7] []
oms-integration.log:2023-10-31 00:15:28.198 [INFO] - [a31f65fd-9127-4dd4-b147-adc24a71198b] [GET] [http://10.186.16.126:8088] [/connector/task/v2/batchDetail] [200] [6] []
oms-integration.log:2023-10-31 00:15:28.214 [INFO] - [a31f65fd-9127-4dd4-b147-adc24a71198b] [GET] [http://10.186.16.126:8088] [/connector/task/v2/batchDetail] [200] [6] []
``` 

在CM的cm-web.log中, 根据trace ID查找调用: 

```
2023-10-31 00:15:28 INFO com.alibaba.drc.controller.StoreMgrAction.listStore(StoreMgrAction.java:764) - [a31f65fd-9127-4dd4-b147-adc24a71198b] list crawler :[Crawler{id=1, subTopicId=1, storeId=1, name='10.186.16.126-7100:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0:0000000001', subTopicName='ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0', port=17000, storePort=7100, dataSource=':', errMsg='[Mon Oct 30 16:31:01 2023] DRC_500:Store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 started successfully
', userData='', role='master', autoWatch=1, status=1, version='1.0', gmtCreated=Wed Oct 25 15:44:51 CST 2023, gmtModified=Tue Oct 31 00:15:24 CST 2023, crawlerStatus=1, filePath='/home/ds/store/store7100/conf/stores.conf', ip='10.186.16.126', location='0'}],topic:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0
``` 

TODO: 只能查到获取状态的请求, 而不能查询到实际store状态的问题, 还需要进一步的方法

## 日志dss-operation.log

![image2023-10-31 15:30:36.png](/assets/01KJBZ3FQT8FKX47BF6Q8C4X7J/image2023-10-31%2015%3A30%3A36.png)

日志信息, 均为包com.alipay.dss.core.operation打印, 含义为: dss(猜测是OMS的内部名), 提供的服务, 服务类型为上图中的service和innerService

# 进程 store

日志列表: 

```
l-wx------ 1 ds ds 64 Oct 31 13:05 1 -> /u01/ds/store/store7100/log/store.log
l-wx------ 1 ds ds 64 Oct 31 13:05 2 -> /u01/ds/store/store7100/log/store.log
l-wx------ 1 ds ds 64 Oct 31 13:05 3 -> /u01/ds/store/store7100/log/congo_0
l-wx------ 1 ds ds 64 Oct 31 13:05 4 -> /u01/ds/store/store7100/queue_log_0
lrwx------ 1 ds ds 64 Oct 31 14:02 7 -> /u01/ds/store/store7100/DLlib/complite.log
l-wx------ 1 ds ds 64 Oct 31 14:02 8 -> /u01/ds/store/store7100/ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001/-.0/queue_log_0
``` 

## 日志store.log

store和子进程(java kafka)的标准流和错误流

## 日志DLlib/complite.log

没有内容

## 日志congo_0

是抓取器的日志 (logminer?), 样例: 

日志中没有trace ID, 日志格式未知 (猜测66990是任务ID)

```
[root@R740-26 log]# grep 66990 *
congo_0:2023-10-31 13:05:43 [NOTICE] [StoresManager.cpp:251 66990,3011479744] Create scheduler with file size 16MB, cache 10MB and flush limit 32MB, compressed 2, , read wait number 0, enable index false, read thread number 2, index arena size 8
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreCommandsProcessor.cpp:107 66990,2841106176] Get Request GET /start?subTopic=ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0&subId=0000000001&store.conf=%2Fhome%2Fds%2Fstore%2Fstore7100%2Fconf%2Fcrawler.conf HTTP/1.1
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2841106176] Request params: store.conf = /home/ds/store/store7100/conf/crawler.conf
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2841106176] Request params: subId = 0000000001
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2841106176] Request params: subTopic = ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0
congo_0:2023-10-31 13:05:44 [NOTICE] [StoresManager.cpp:343 66990,2841106176] Starting store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 with config /home/ds/store/store7100/conf/crawler.conf
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:561 66990,2841106176] sectionnames size:13
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:577 66990,2841106176] load 0th sectionnames:global
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:config.version value:3
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:running.mode value:strict
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:mysql2store is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:store2store is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:ob2store is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:577 66990,2841106176] load 4th sectionnames:store
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:clearer.outdated value:432000
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:clearer.period value:3600
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:client.wait value:43200
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:connection.numLimit value:100
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:drcnetListenPort value:17001
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:listeningPort value:17000
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:reader value:on
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:repStatus value:master
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:storesManager.port value:7100
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:writer.threshold value:1
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:writer.type value:message
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:liboblog is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:partition is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:hbase2store is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:oracle2store is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:577 66990,2841106176] load 9th sectionnames:deliver2store
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:connect.deliver_proterties_path value:./kafka/config/connect-drcdeliver.properties
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:error.level value:WARN
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.connector.class value:com.taobao.drc.logminer.connect.LogMinerConnector
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.filter.owner_black_list value:^SYS$,^SYSTEM$,^APPQOSSYS$,^SYSMAN$,^DBSNMP$,^WMSYS$,^XDB$,^ORDDATA$,^MDSYS$,^APEX_030200$,^CTXSYS$,^EXFSYS$,^LBACSYS$
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.filter.owner_white_list value:.*
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.full_table_name_black_list value:ANONYMOUS.*|APEX_030200.*|APEX_040200.*|APEX_PUBLIC_USER.*|APPQOSSYS.*|AUDSYS.*|CTXSYS.*|DBSNMP.*|DIP.*|DMSYS.*|DVF.*|DVSYS.*|EXFSYS.*|FLOWS_30000.*|FLOWS_FILES.*|GSMADMIN_INTERNAL.*|GSMCATUSER.*|GSMUSER.*|LBACSYS.*|MDDATA.*|MDSYS.*|MGMT_VIEW.*|OJVMSYS.*|OLAPSYS.*|OORACLE_OCM.*|ORACLE_OCM.*|ORDDATA.*|ORDPLUGINS.*|ORDSYS.*|OUTLN.*|OWBSYS.*|OWBSYS_AUDIT.*|SI_INFORMTN_SCHEMA.*|SPATIAL_CSW_ADMIN_USR.*|SPATIAL_WFS_ADMIN_USR.*|SYS.*|SYSBACKUP.*|SYSDG.*|SYSKM.*|SYSMAN.*|SYSTEM.*|TSMSYS.*|WKPROXY.*|WKSYS.*|WK_TEST.*|WMSYS.*|XDB.*|XS$NULL.*
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.full_table_name_white_list value:OBTEST1.T1
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.name value:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0-1
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.oracle.password value:111111
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.oracle.url value:10.186.16.126:1521/EE.ORACLE.DOCKER
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.oracle.user value:OBTEST
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.output_rowid value:true
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.session_timezone value:+08:00
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:logminer.use_independent_fetcher_per_instance value:true
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:master.binlog value:1611430
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:master.offset value:1
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:master.timestamp value:1698228006
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:parallelism value:32
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:pipeline value:reset,read,parse,filter,consume
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:subId value:0000000001
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:subTopic value:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:subreader.bootstrup value:./kafka/bin/connect-drcdeliver.sh
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:subreader.type value:logminer
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:581 66990,2841106176] load config key:topic value:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:unit is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:db2tostore is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:567 66990,2841106176] Config of sectionnames:logproxy2store is empty
congo_0:2023-10-31 13:05:44 [NOTICE] [StoreManager.cpp:823 66990,2841106176] succeed to init record sent log with log.sent.record=0, log.sent.record.users=, log.sent.max.files=50
congo_0:2023-10-31 13:06:27 [WARN] [StoreManager.cpp:669 66990,2841106176] Load Rule Library : library Name not found, FilterRule not work
congo_0:2023-10-31 13:06:27 [NOTICE] [StoreManager.cpp:1369 66990,2841106176] drcnet server buf size: 1048576
congo_0:2023-10-31 13:06:28 [WARN] [StoreManager.cpp:1380 66990,2841106176] drcnet http listen port is 17001
congo_0:2023-10-31 13:06:28 [NOTICE] [StoreManager.cpp:1473 66990,2841106176] Store Start use normal  mode
congo_0:2023-10-31 13:06:28 [NOTICE] [StoreManager.cpp:1973 66990,2558494464] Store restart with 1698228006
congo_0:2023-10-31 13:06:28 [NOTICE] [StoresManager.cpp:414 66990,2841106176] Store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 first start
congo_0:2023-10-31 13:06:28 [NOTICE] [StoresManager.cpp:424 66990,2841106176] Store quota remains 9 out of 10 after start, 1 store instance in memory
congo_0:2023-10-31 13:06:28 [NOTICE] [Reader.cpp:128 66990,2558494464] Drc Frame ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 started
congo_0:2023-10-31 13:06:29 [NOTICE] [Reader.cpp:90 66990,2558494464] register module deliver2store ok
congo_0:2023-10-31 13:06:29 [NOTICE] [DrcModRunner.cpp:166 66990,2541709056] pipeline diagnose off
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:87 66990,2541709056] get checkpoint info from queue is 1611430, 0, 1698228006, 0
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:89 66990,2541709056] get checkpoint info from config is 1611430, 1, 1698228006
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:99 66990,2541709056] set deliver checkpoint: &mCurCheckpoint:[FileID: 1611430][FileOffset: 0][TimeStamp: 1698228006]
congo_0:2023-10-31 13:06:29 [NOTICE] [subreadermanager.cpp:38 66990,2541709056] begin to start deliver2store's sub-reader: ./kafka/bin/connect-drcdeliver.sh
congo_0:2023-10-31 13:06:29 [NOTICE] [subreadermanager.cpp:164 66990,2541709056] sub reader type: logminer
congo_0:2023-10-31 13:06:29 [NOTICE] [subreadermanager.cpp:106 66990,2541709056] DELIVER_LISTEN_PORT is 40027
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property clearer.outdated=432000 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property clearer.period=3600 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property client.wait=43200 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property connect.deliver_proterties_path=./kafka/config/connect-drcdeliver.properties when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property connection.numLimit=100 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property drcnetListenPort=17001 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property error.level=WARN when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property listeningPort=17000 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property master.binlog=1611430 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property master.offset=1 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property master.timestamp=1698228006 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property network.listen_port=40027 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property parallelism=32 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property pipeline=reset,read,parse,filter,consume when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property reader=on when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property repStatus=master when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property stores.listen.port=7100 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property storesManager.port=7100 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property subreader.bootstrup=./kafka/bin/connect-drcdeliver.sh when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property subreader.profile_path= when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property subreader.type=logminer when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property writer.threshold=1 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [WARN] [subreadermanager.cpp:69 66990,2541709056] skipped property writer.type=message when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:172 66990,2541709056] deliver2store initModule ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:29 [NOTICE] [deliver2store.cpp:198 66990,2541709056] deliver2store initDrcPack ok
congo_0:2023-10-31 13:06:31 [WARN] [stores.cpp:95 66990,3011479744] reset sigHandle for signal [SIGTERM]
congo_0:2023-10-31 13:06:31 [WARN] [stores.cpp:101 66990,3011479744] reset sigHandle for signal [SIGINT]
congo_0:2023-10-31 13:08:35 [NOTICE] [StoreCommandsProcessor.cpp:107 66990,2832713472] Get Request POST /error/report HTTP/1.1
congo_0:2023-10-31 13:08:35 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2832713472] Request params: content = {"level":"FATAL","gmt":1698728915,"code":"1","message":"Oracle log file does not exist","reason":"The Oracle log file containing scn 1611430 does not exist in Oracle instance 1","proposal":"Recover the Oracle log file or re-create the migration task","context":{"instance":"1","timestamp_or_scn_key":"scn","timestamp_or_scn_value":"1611430"}}
congo_0:2023-10-31 13:08:37 [ERROR] [deliver2store.cpp:342 66990,2508138240] sub reader is exited, we should terminate
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:204 66990,2541709056] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:178 66990,2541709056] deliver2store cleanupModule() called
congo_0:2023-10-31 13:08:37 [NOTICE] [deliver2store.cpp:188 66990,2541709056] after deliver2store cleanupModule() called
congo_0:2023-10-31 13:34:14 [NOTICE] [StoreCommandsProcessor.cpp:107 66990,2533316352] Get Request GET /clear?subTopic=ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0&subId=0000000001 HTTP/1.1
congo_0:2023-10-31 13:34:14 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2533316352] Request params: subId = 0000000001
congo_0:2023-10-31 13:34:14 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2533316352] Request params: subTopic = ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0
congo_0:2023-10-31 13:34:14 [NOTICE] [StoresManager.cpp:503 66990,2533316352] Clear store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001
congo_0:2023-10-31 13:34:14 [NOTICE] [StoresManager.cpp:452 66990,2533316352] Stop store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001
congo_0:2023-10-31 13:34:26 [NOTICE] [StoresManager.cpp:545 66990,2533316352] Store quota remains 10 out of 10 after clear
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreCommandsProcessor.cpp:107 66990,2575279872] Get Request GET /start?subTopic=ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0&subId=0000000001&store.conf=%2Fhome%2Fds%2Fstore%2Fstore7100%2Fconf%2Fcrawler.conf HTTP/1.1
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2575279872] Request params: store.conf = /home/ds/store/store7100/conf/crawler.conf
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2575279872] Request params: subId = 0000000001
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2575279872] Request params: subTopic = ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0
congo_0:2023-10-31 13:35:03 [NOTICE] [StoresManager.cpp:343 66990,2575279872] Starting store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 with config /home/ds/store/store7100/conf/crawler.conf
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:561 66990,2575279872] sectionnames size:13
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:577 66990,2575279872] load 0th sectionnames:global
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:config.version value:3
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:running.mode value:strict
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:mysql2store is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:store2store is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:ob2store is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:577 66990,2575279872] load 4th sectionnames:store
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:clearer.outdated value:432000
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:clearer.period value:3600
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:client.wait value:43200
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:connection.numLimit value:100
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:drcnetListenPort value:17002
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:listeningPort value:17001
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:reader value:on
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:repStatus value:master
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:storesManager.port value:7100
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:writer.threshold value:1
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:writer.type value:message
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:liboblog is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:partition is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:hbase2store is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:oracle2store is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:577 66990,2575279872] load 9th sectionnames:deliver2store
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:connect.deliver_proterties_path value:./kafka/config/connect-drcdeliver.properties
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:error.level value:WARN
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.connector.class value:com.taobao.drc.logminer.connect.LogMinerConnector
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.filter.owner_black_list value:^SYS$,^SYSTEM$,^APPQOSSYS$,^SYSMAN$,^DBSNMP$,^WMSYS$,^XDB$,^ORDDATA$,^MDSYS$,^APEX_030200$,^CTXSYS$,^EXFSYS$,^LBACSYS$
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.filter.owner_white_list value:.*
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.full_table_name_black_list value:ANONYMOUS.*|APEX_030200.*|APEX_040200.*|APEX_PUBLIC_USER.*|APPQOSSYS.*|AUDSYS.*|CTXSYS.*|DBSNMP.*|DIP.*|DMSYS.*|DVF.*|DVSYS.*|EXFSYS.*|FLOWS_30000.*|FLOWS_FILES.*|GSMADMIN_INTERNAL.*|GSMCATUSER.*|GSMUSER.*|LBACSYS.*|MDDATA.*|MDSYS.*|MGMT_VIEW.*|OJVMSYS.*|OLAPSYS.*|OORACLE_OCM.*|ORACLE_OCM.*|ORDDATA.*|ORDPLUGINS.*|ORDSYS.*|OUTLN.*|OWBSYS.*|OWBSYS_AUDIT.*|SI_INFORMTN_SCHEMA.*|SPATIAL_CSW_ADMIN_USR.*|SPATIAL_WFS_ADMIN_USR.*|SYS.*|SYSBACKUP.*|SYSDG.*|SYSKM.*|SYSMAN.*|SYSTEM.*|TSMSYS.*|WKPROXY.*|WKSYS.*|WK_TEST.*|WMSYS.*|XDB.*|XS$NULL.*
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.full_table_name_white_list value:OBTEST1.T1
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.name value:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0-1
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.oracle.password value:111111
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.oracle.url value:10.186.16.126:1521/EE.ORACLE.DOCKER
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.oracle.user value:OBTEST
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.output_rowid value:true
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.session_timezone value:+08:00
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.use_independent_fetcher_per_instance value:true
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:master.binlog value:1611430
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:master.offset value:1
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:master.timestamp value:1698228006
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:parallelism value:32
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:pipeline value:reset,read,parse,filter,consume
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:subId value:0000000001
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:subTopic value:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:subreader.bootstrup value:./kafka/bin/connect-drcdeliver.sh
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:subreader.type value:logminer
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:topic value:ORACLE_np_56el1g1jvrs0_56el4eyvwvxc
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:unit is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:db2tostore is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:567 66990,2575279872] Config of sectionnames:logproxy2store is empty
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:823 66990,2575279872] succeed to init record sent log with log.sent.record=0, log.sent.record.users=, log.sent.max.files=50
congo_0:2023-10-31 13:35:29 [WARN] [StoreManager.cpp:669 66990,2575279872] Load Rule Library : library Name not found, FilterRule not work
congo_0:2023-10-31 13:35:29 [NOTICE] [StoreManager.cpp:1369 66990,2575279872] drcnet server buf size: 1048576
congo_0:2023-10-31 13:35:30 [WARN] [StoreManager.cpp:1380 66990,2575279872] drcnet http listen port is 17002
congo_0:2023-10-31 13:35:30 [NOTICE] [StoreManager.cpp:1473 66990,2575279872] Store Start use normal  mode
congo_0:2023-10-31 13:35:30 [NOTICE] [StoreManager.cpp:1973 66990,2734675712] Store restart with 1698228006
congo_0:2023-10-31 13:35:30 [NOTICE] [Reader.cpp:128 66990,2734675712] Drc Frame ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 started
congo_0:2023-10-31 13:35:30 [NOTICE] [Reader.cpp:90 66990,2734675712] register module deliver2store ok
congo_0:2023-10-31 13:35:30 [NOTICE] [DrcModRunner.cpp:166 66990,2717890304] pipeline diagnose off
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:87 66990,2717890304] get checkpoint info from queue is 1611430, 0, 1698228006, 0
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:89 66990,2717890304] get checkpoint info from config is 1611430, 1, 1698228006
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:99 66990,2717890304] set deliver checkpoint: &mCurCheckpoint:[FileID: 1611430][FileOffset: 0][TimeStamp: 1698228006]
congo_0:2023-10-31 13:35:30 [NOTICE] [subreadermanager.cpp:38 66990,2717890304] begin to start deliver2store's sub-reader: ./kafka/bin/connect-drcdeliver.sh
congo_0:2023-10-31 13:35:30 [NOTICE] [subreadermanager.cpp:164 66990,2717890304] sub reader type: logminer
congo_0:2023-10-31 13:35:30 [NOTICE] [subreadermanager.cpp:106 66990,2717890304] DELIVER_LISTEN_PORT is 39865
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property clearer.outdated=432000 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property clearer.period=3600 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property client.wait=43200 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property connect.deliver_proterties_path=./kafka/config/connect-drcdeliver.properties when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property connection.numLimit=100 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property drcnetListenPort=17002 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property error.level=WARN when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property listeningPort=17001 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property master.binlog=1611430 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property master.offset=1 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property master.timestamp=1698228006 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property network.listen_port=39865 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property parallelism=32 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property pipeline=reset,read,parse,filter,consume when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property reader=on when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property repStatus=master when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property stores.listen.port=7100 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property storesManager.port=7100 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property subreader.bootstrup=./kafka/bin/connect-drcdeliver.sh when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property subreader.profile_path= when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property subreader.type=logminer when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property writer.threshold=1 when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [WARN] [subreadermanager.cpp:69 66990,2717890304] skipped property writer.type=message when building deliver2store config, property name must starts with logminer
congo_0:2023-10-31 13:35:30 [NOTICE] [StoresManager.cpp:414 66990,2575279872] Store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 first start
congo_0:2023-10-31 13:35:30 [NOTICE] [StoresManager.cpp:424 66990,2575279872] Store quota remains 9 out of 10 after start, 1 store instance in memory
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:172 66990,2717890304] deliver2store initModule ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:30 [NOTICE] [deliver2store.cpp:198 66990,2717890304] deliver2store initDrcPack ok
congo_0:2023-10-31 13:35:33 [WARN] [stores.cpp:95 66990,3011479744] reset sigHandle for signal [SIGTERM]
congo_0:2023-10-31 13:35:33 [WARN] [stores.cpp:101 66990,3011479744] reset sigHandle for signal [SIGINT]
congo_0:2023-10-31 13:36:13 [NOTICE] [StoreCommandsProcessor.cpp:107 66990,2508138240] Get Request POST /error/report HTTP/1.1
congo_0:2023-10-31 13:36:13 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2508138240] Request params: content = {"level":"FATAL","gmt":1698730573,"code":"1","message":"Oracle log file does not exist","reason":"The Oracle log file containing scn 1611430 does not exist in Oracle instance 1","proposal":"Recover the Oracle log file or re-create the migration task","context":{"instance":"1","timestamp_or_scn_key":"scn","timestamp_or_scn_value":"1611430"}}
congo_0:2023-10-31 13:36:15 [ERROR] [deliver2store.cpp:342 66990,2566887168] sub reader is exited, we should terminate
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:204 66990,2717890304] deliver2store cleanupDrcPack() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:178 66990,2717890304] deliver2store cleanupModule() called
congo_0:2023-10-31 13:36:15 [NOTICE] [deliver2store.cpp:188 66990,2717890304] after deliver2store cleanupModule() called
grep: connector: Is a directory
[root@R740-26 log]#
``` 

关键日志样例: 

```
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreCommandsProcessor.cpp:107 66990,2575279872] Get Request GET /start?subTopic=ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0&subId=0000000001&store.conf=%2Fhome%2Fds%2Fstore%2Fstore7100%2Fconf%2Fcrawler.conf HTTP/1.1

...
 
congo_0:2023-10-31 13:35:03 [NOTICE] [StoresManager.cpp:343 66990,2575279872] Starting store ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001 with config /home/ds/store/store7100/conf/crawler.conf

...
 
congo_0:2023-10-31 13:35:03 [NOTICE] [StoreManager.cpp:581 66990,2575279872] load config key:logminer.full_table_name_black_list value:ANONYMOUS.*|APEX_030200.*|APEX_040200.*|APEX_PUBLIC_USER.*|APPQOSSYS.*|AUDSYS.*|CTXSYS.*|DBSNMP.*|DIP.*|DMSYS.*|DVF.*|DVSYS.*|EXFSYS.*|FLOWS_30000.*|FLOWS_FILES.*|GSMADMIN_INTERNAL.*|GSMCATUSER.*|GSMUSER.*|LBACSYS.*|MDDATA.*|MDSYS.*|MGMT_VIEW.*|OJVMSYS.*|OLAPSYS.*|OORACLE_OCM.*|ORACLE_OCM.*|ORDDATA.*|ORDPLUGINS.*|ORDSYS.*|OUTLN.*|OWBSYS.*|OWBSYS_AUDIT.*|SI_INFORMTN_SCHEMA.*|SPATIAL_CSW_ADMIN_USR.*|SPATIAL_WFS_ADMIN_USR.*|SYS.*|SYSBACKUP.*|SYSDG.*|SYSKM.*|SYSMAN.*|SYSTEM.*|TSMSYS.*|WKPROXY.*|WKSYS.*|WK_TEST.*|WMSYS.*|XDB.*|XS$NULL.*

...
congo_0:2023-10-31 13:36:13 [NOTICE] [StoreCommandsProcessor.cpp:112 66990,2508138240] Request params: content = {"level":"FATAL","gmt":1698730573,"code":"1","message":"Oracle log file does not exist","reason":"The Oracle log file containing scn 1611430 does not exist in Oracle instance 1","proposal":"Recover the Oracle log file or re-create the migration task","context":{"instance":"1","timestamp_or_scn_key":"scn","timestamp_or_scn_value":"1611430"}}
congo_0:2023-10-31 13:36:15 [ERROR] [deliver2store.cpp:342 66990,2566887168] sub reader is exited, we should terminate

``` 

## 日志queue_log_0

queue的管理服务? 日志样例: 

```
2023-10-31 13:35:03 [NOTICE] [queue_service.cc:74 66990,2575279872] queue service started
2023-10-31 13:35:29 [INFO] [scheduler.cc:188 66990,2575279872] adding DrcQueue ./ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001/-.0
2023-10-31 13:35:29 [INFO] [scheduler.cc:234 66990,2575279872] restarting flush thread
2023-10-31 13:35:29 [ERROR] [scheduler.cc:240 66990,2575279872] queue remains
``` 

猜测: 66990作为任务ID

## 日志 ORACLE_np_56el1g1jvrs0_56el4eyvwvxc-1-0.0000000001/-.0/queue_log_0

单个任务的queue管理日志, 样例: 

```
2023-10-31 13:08:37 [INFO] [unit_queue.cc:185 66990,2866284288] UnitQueue 1398:  clear buffer
2023-10-31 13:08:37 [INFO] [drc_queue.cc:749 66990,2541709056] Force Flush Blocking: success
2023-10-31 13:08:37 [INFO] [unit_queue.cc:508 66990,2558494464] UnitQueue 1398:  read_ref 0 -> 1, GetLastInfo
2023-10-31 13:08:37 [INFO] [unit_queue.cc:577 66990,2558494464] UnitQueue 1398:  DiskOnly -> Reading
2023-10-31 13:08:37 [INFO] [unit_queue.cc:508 66990,2558494464] UnitQueue 1398:  read_ref 1 -> 2, Read
2023-10-31 13:08:37 [INFO] [unit_queue.cc:213 66990,2558494464] UnitQueue 1398: reading local for all data
2023-10-31 13:08:37 [INFO] [unit_queue.cc:577 66990,2558494464] UnitQueue 1398:  Reading -> Caching
2023-10-31 13:08:37 [INFO] [unit_queue.cc:508 66990,2558494464] UnitQueue 1398:  read_ref 2 -> 1, Read
2023-10-31 13:08:37 [INFO] [unit_queue.cc:508 66990,2558494464] UnitQueue 1398:  read_ref 1 -> 0, GetLastInfo
2023-10-31 13:08:37 [INFO] [unit_queue.cc:577 66990,2558494464] UnitQueue 1398:  Caching -> DiskOnly
2023-10-31 13:08:37 [INFO] [unit_queue.cc:185 66990,2558494464] UnitQueue 1398:  clear buffer
2023-10-31 13:13:43 [INFO] [drc_queue.cc:1204 66990,2857891584] Timed Flush
 
...
 
2023-10-31 14:35:32 [INFO] [drc_queue.cc:846 66990,2533316352] DrcQueue deleter quit earlier for met a UnitQueue with ref count 1
2023-10-31 14:35:32 [INFO] [drc_queue.cc:890 66990,2533316352] 1379 UnitQueues deleted by user-deletion(timestamp = 1698302129, record_id = 0, file_id = 0, file_pos = 0): from 22 to 1400
``` 

猜测: 66990作为任务ID

问题: 存在1400个unit queue? 

# 进程supervisor

日志列表: 

```
l-wx------ 1 ds ds 64 Oct 30 22:26 1 -> /u01/ds/supervisor/log/out.log
l-wx------ 1 ds ds 64 Oct 30 22:26 10 -> /home/admin/logs/supervisor/error.log
l-wx------ 1 ds ds 64 Oct 30 22:26 11 -> /home/admin/logs/supervisor/legacy.log
l-wx------ 1 ds ds 64 Oct 30 22:26 12 -> /home/admin/logs/supervisor/routine.log
l-wx------ 1 ds ds 64 Oct 30 22:26 13 -> /home/admin/logs/supervisor/command.log
l-wx------ 1 ds ds 64 Oct 30 22:26 14 -> /home/admin/logs/supervisor/dbcat.log
l-wx------ 1 ds ds 64 Oct 30 17:35 2 -> /u01/ds/supervisor/log/out.log
l-wx------ 1 ds ds 64 Oct 30 22:26 22 -> /u01/ds/supervisor/data/async_command/LOG
lr-x------ 1 ds ds 64 Oct 30 22:26 24 -> /u01/ds/supervisor/data/async_command
lrwx------ 1 ds ds 64 Oct 30 22:26 25 -> /u01/ds/supervisor/data/async_command/LOCK
lr-x------ 1 ds ds 64 Oct 30 22:26 26 -> /u01/ds/supervisor/data/async_command
l-wx------ 1 ds ds 64 Oct 30 22:26 27 -> /u01/ds/supervisor/data/async_command/000014.log
l-wx------ 1 ds ds 64 Oct 30 22:26 28 -> /u01/ds/supervisor/data/async_command/MANIFEST-000015
l-wx------ 1 ds ds 64 Oct 30 22:26 3 -> /u01/ds/supervisor/log/gc.log
l-wx------ 1 ds ds 64 Oct 30 22:26 8 -> /home/admin/logs/supervisor/supervisor.log
l-wx------ 1 ds ds 64 Oct 30 22:26 9 -> /home/admin/logs/supervisor/monitor.log
``` 

日志配置: classpath:logback.xml: 

```
<?xml version="1.0"?>
<configuration>
    <property file="${configDir:-/home/ds/supervisor/config/}drc.properties" />

    <springProperty scope="context" name="logging.path" source="logging.path" defaultValue="/home/admin/logs/supervisor" />
    <springProperty scope="context" name="logging.level" source="logging.level" defaultValue="INFO" />

    <appender name="CONSOLE-APPENDER" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}][%p][%t][%c{0}:%line] %m%n</pattern>
        </encoder>
    </appender>

    <appender name="DEFAULT-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>${logging.path}/supervisor.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${logging.path}/supervisor.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>7</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}][%p][%t][%c{0}:%line][%X{traceId}] %m%n</pattern>
        </encoder>
    </appender>

    <appender name="MONITOR-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>${logging.path}/monitor.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${logging.path}/monitor.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>7</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}][%X{traceId}] %m%n</pattern>
        </encoder>
    </appender>

    <appender name="ERROR-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>error</level>
        </filter>
        <file>${logging.path}/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${logging.path}/error.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>7</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}][%p][%t][%c{0}:%line][%X{traceId}] %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <appender name="LEGACY-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>${logging.path}/legacy.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${logging.path}/legacy.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>7</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}][%p][%t][%c{0}:%line][%X{traceId}] %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <appender name="ROUTINE-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>${logging.path}/routine.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${logging.path}/routine.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>3</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}][%p][%t][%c{0}:%line][%X{traceId}] %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <appender name="COMMAND-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>${logging.path}/command.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${logging.path}/command.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>7</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}][%p][%t][%c{0}:%line][%X{traceId}][%X{TAG}] %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <appender name="DBCAT-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>${logging.path}/dbcat.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${logging.path}/dbcat.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>3</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{20} - [%X{traceId}] %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <logger name="ConsoleLogger" level="${logging.level}" additivity="false">
        <appender-ref ref="CONSOLE-APPENDER"/>
    </logger>

    <logger name="MONITOR" level="${logging.level}" additivity="false">
        <appender-ref ref="MONITOR-APPENDER"/>
    </logger>

    <logger name="com.alibaba.drc.job" level="${logging.level}" additivity="false">
        <appender-ref ref="ROUTINE-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.oceanbase.oms.supervisor.scavenge" level="${logging.level}" additivity="false">
        <appender-ref ref="ROUTINE-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.oceanbase.oms.supervisor.background" level="${logging.level}" additivity="false">
        <appender-ref ref="ROUTINE-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.oceanbase.oms.supervisor.command" level="${logging.level}" additivity="false">
        <appender-ref ref="COMMAND-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.alibaba.drc" level="${logging.level}" additivity="false">
        <appender-ref ref="LEGACY-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </logger>

    <logger name="com.oceanbase.obtools" level="${logging.level}" additivity="false">
        <appender-ref ref="DBCAT-APPENDER"/>
    </logger>

    <root level="${logging.level}">
        <appender-ref ref="DEFAULT-APPENDER"/>
        <appender-ref ref="ERROR-APPENDER"/>
    </root>
</configuration>
``` 

## dbcat.log

TODO: 数据结构获取是从supervisor进行? 

# 进程 com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap (回放端)

```
l-wx------ 1 ds ds 64 Nov  6 00:15 1 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002.out
l-wx------ 1 ds ds 64 Nov  6 00:15 10 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/connector_source_msg.log
l-wx------ 1 ds ds 64 Nov  6 00:15 11 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/connector.log
l-wx------ 1 ds ds 64 Nov  6 00:15 12 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/connector_sink_msg.log
l-wx------ 1 ds ds 64 Nov  6 00:15 13 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/connector_filter_msg.log
l-wx------ 1 ds ds 64 Nov  6 00:15 14 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/ddl_msg.log
l-wx------ 1 ds ds 64 Nov  6 00:15 15 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/manual_table.log
l-wx------ 1 ds ds 64 Nov  6 00:15 17 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/error.log
l-wx------ 1 ds ds 64 Nov  6 00:15 19 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/trace.log
l-wx------ 1 ds ds 64 Nov  6 00:01 2 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002.out
lrwx------ 1 ds ds 64 Nov  6 00:15 26 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/connector.pid
l-wx------ 1 ds ds 64 Nov  6 00:15 8 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/metrics.log
l-wx------ 1 ds ds 64 Nov  6 00:15 9 -> /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/sql_msg.log
``` 

日志配置: 

/u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/jdbc_connector.jar:logback.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_ROOT" value="${log.file.root:-./logs}" />
    <property name="LOG_LEVEL" value="${log.level:-info}" />

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%p] [%t] [%m]%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <appender name="METRICS" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/msg/metrics.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/msg/metrics.%d{yyyy-MM-dd}.log.gz</FileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] %m%n</pattern>
        </layout>
    </appender>

    <appender name="SQL-MSG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/msg/sql_msg.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/msg/sql_msg.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] %m%n</pattern>
        </layout>
    </appender>

    <appender name="TRACE-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/trace.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/trace.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] %m%n</pattern>
        </layout>
    </appender>

    <appender name="SOURCE-MSG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/msg/connector_source_msg.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/msg/connector_source_msg.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%t] %m%n</pattern>
        </layout>
    </appender>

    <appender name="SINK-MSG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/msg/connector_sink_msg.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/msg/connector_sink_msg.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%t] %m%n</pattern>
        </layout>
    </appender>

    <appender name="FILTER-MSG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/msg/connector_filter_msg.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/msg/connector_filter_msg.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%t] %m%n</pattern>
        </layout>
    </appender>

    <appender name="DDL-MSG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/msg/ddl_msg.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/msg/ddl_msg.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%t] %m%n</pattern>
        </layout>
    </appender>

    <appender name="MANUAL-TABLE" class="ch.qos.logback.core.FileAppender">
        <File>${LOG_ROOT}/msg/manual_table.log</File>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%t] %m%n</pattern>
        </layout>
    </appender>

    <logger name="sql-msg" level="info" additivity="false">
        <appender-ref ref="SQL-MSG" />
    </logger>

    <logger name="ddl-msg" level="info" additivity="false">
        <appender-ref ref="DDL-MSG" />
    </logger>

    <logger name="source-msg" level="${LOG_LEVEL}" additivity="false">
        <appender-ref ref="SOURCE-MSG" />
    </logger>

    <logger name="sink-msg" level="${LOG_LEVEL}" additivity="false">
        <appender-ref ref="SINK-MSG" />
    </logger>

    <logger name="filter-msg" level="${LOG_LEVEL}" additivity="false">
        <appender-ref ref="FILTER-MSG" />
    </logger>

    <logger name="trace-log" level="${LOG_LEVEL}" additivity="false">
        <appender-ref ref="TRACE-LOG" />
    </logger>

    <logger name="metrics-msg" level="info" additivity="false">
        <appender-ref ref="METRICS"/>
    </logger>

    <logger name="manual-table" level="info" additivity="false">
        <appender-ref ref="MANUAL-TABLE" />
    </logger>

    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/connector.log</File>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/connector.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%p] [%t] [%m]%n</pattern>
        </layout>
    </appender>

    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_ROOT}/error.log</File>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_ROOT}/error.%d{yyyy-MM-dd_HH}.log.gz</FileNamePattern>
            <maxHistory>168</maxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyy-MM-dd HH:mm:ss.SSS}] [%p] [%t] [%m]%n</pattern>
        </layout>
    </appender>

    <Root level="${LOG_LEVEL}" additivity="false">
        <appender-ref ref="INFO"/>
        <appender-ref ref="ERROR"/>
    </Root>

</configuration>

``` 

## 日志辅助类

  - source-msg, sink-msg, filter-msg
    - 打印点位: com.oceanbase.oms.connector.common.log.MsgUtils
  - sql-msg, ddl-msg, metrics, manual_table, trace
    - 打印点位: com.oceanbase.oms.connector.common.log.Log

## 日志 /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/trace.log

打印辅助函数: RecordBatchUtil.invokeBatchCallBack

![image2023-11-6 23:4:6.png](/assets/01KJBZ3FQT8FKX47BF6Q8C4X7J/image2023-11-6%2023%3A4%3A6.png)

在recordBatch中, 增加trace, 并在处理完成后, 打印到trace.log

举例: 

```
[2023-11-06 23:20:02.425] {"id":1385,"totalCost":165424,"batchSize":0,"memory":0,"batchType":"TRANSACTION","items":[{"span":"batch","step":"create","timeMs":1699283837000,"otherInfo":null},{"span":"serial-process","step":"entry","timeMs":1699284002424,"otherInfo":null},{"span":"serial-process","step":"callback-processed","timeMs":1699284002424,"otherInfo":null},{"span":"record-batch-listener","step":"consume-done","timeMs":1699284002424,"otherInfo":null}]}
``` 

FIXME:

  1. 需要根据record的某种条件, 确定打印哪个记录的trace
  2. trace覆盖的范围太窄, 仅包括少数几个步骤

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/connector_source_msg.log

代码位置: com.oceanbase.oms.connector.batch.CommonTransactionAssembler.TransactionAssemblerInner#putTransaction

意义: 

  - 进程: com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap (回放段)
  - 位置: 第一个处理器, 将数据处理完后, 送入第一个输出队列之前, 将数据打印出来

注意: 调整 com.oceanbase.oms.connector.common.log.MsgSegment logger的级别为DEBUG, 可以打印出数据本身 (需测试)

样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析] 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/connector_sink_msg.log

代码位置: com.oceanbase.oms.connector.jdbc.sink.Writer

意义: 

  - 进程: com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap (回放段)
  - 位置: 最后一个处理器, 将数据回放完后, 将数据和指标(affect, 回放时间等) 打印出来  
  

注意: 调整 com.oceanbase.oms.connector.common.log.MsgSegment logger的级别为DEBUG, 可以打印出数据本身 (需测试). 但打印代码写错了: 

![image2023-11-8 11:36:53.png](/assets/01KJBZ3FQT8FKX47BF6Q8C4X7J/image2023-11-8%2011%3A36%3A53.png)

样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析] 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/connector_filter_msg.log

代码位置: 

  - com.oceanbase.oms.connector.transformer.RowFilterTransformer
  - com.oceanbase.oms.connector.transformer.DDLTransformer

意义: 

  - 进程: com.oceanbase.oms.connector.jdbc.coordinator.Bootstrap (回放段)
  - 线程: ETLBatchProcessor
  - 在transformer处理完数据后, 打印数据

注意: 调整 com.oceanbase.oms.connector.common.log.MsgSegment logger的级别为DEBUG, 可以打印出数据本身 (需测试)

TODO:样例: 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/ddl_msg.log

代码位置: 

  - com.oceanbase.oms.connector.jdbc.sink.Writer#beforeDDL
    - 记录在处理DDL之前, 需要进行的 set current DB/schema 的SQL ("set sql:{}, recordDb:{}")
  - com.oceanbase.oms.connector.jdbc.sink.Writer#writeDDL
    - 回放DDL后, 进行记录 ("execute sql:{}, checkpoint [{}] done")
  - com.oceanbase.oms.connector.transformer.DDLTransformer#transform
    - 被黑名单或者过滤条件 过滤的DDL, 会打印日志
    - "receive ddl:{}, checkpoint[{}]": 接收到DDL
    - "skip ddl:{}, checkpoint [{}]": 过滤DDL
    - "filtered ddl:{}, checkpoint [{}]": 过滤DDL
  - LoggerDelegate.registerLogger(new OmsLogger())
    - 不确定作用

样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析] 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/manual_table.log

代码位置: 

  - com.oceanbase.oms.connector.error.AbstractErrorHandler
    - 数据处理过程中, 报错时
      - 如果判断错误需要重试, 打印日志: "retryRecord:{}"
      - 如果判断错误需要跳过, 打印日志: "skipRecord:{}"
  - com.oceanbase.oms.connector.transformer.ddl.CompensateDDLGenerator#getCompensateDDL
    - 完善DDL, 打印日志 "manualTable name:[{}]". (意义不明, 只打印表名)

manual_table这个名字的含义不明

样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析] 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/sql_msg.log

代码位置: 

  - com.oceanbase.oms.connector.jdbc.sink.Writer#batchFlushDML
    - 如果logger (com.oceanbase.oms.connector.jdbc.sink.Writer) 的级别是DEBUG, 在数据回放之前, 会将数据打印出来, "execute:{}"
  - com.oceanbase.oms.connector.jdbc.sink.Writer#realWriteDML
    - 执行DML后, 如果超过了慢SQL阈值, 或者logger (com.oceanbase.oms.connector.jdbc.sink.Writer) 的级别是DEBUG, 会将数据和回放时间打印出来, "execute:{}, \nconsume:{}ms"
  - com.oceanbase.oms.connector.jdbc.sink.Writer#writeDDL
    - 执行DDL后, 如果logger (com.oceanbase.oms.connector.jdbc.sink.Writer) 的级别是DEBUG, 会将数据和回放时间打印出来, "execute:{}, \nconsume:{}ms"

样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析] 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/msg/metrics.log

代码位置: 

  - com.oceanbase.connector.framework.report.ScheduledLogReporterTask
    - 周期性打印信息, 信息函数: metricsInfoStr
    - 样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析] 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/connector.log

标准流

样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析] 

## 日志: /u01/ds/run/10.186.16.126-9000:connector_v2:np_57fdqtrsqiao-incr_trans-1-0:0003000002/logs/error.log

错误流

样例参考: [20231105 - OMS 建立增量复制链路后, 对日志进行分析]
