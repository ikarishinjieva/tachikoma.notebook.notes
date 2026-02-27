---
title: 20221201 - OMS监控
confluence_page_id: 2130278
created_at: 2022-12-01T11:04:59+00:00
updated_at: 2022-12-26T05:57:34+00:00
---

# InfluxDB

influxDB中有部分监控, 监控项: 

["checker.dest.rps"]  
["checker.dest.rt"]  
["checker.dest.write.iops"]  
["checker.source.read.iops"]  
["checker.source.rps"]  
["checker.source.rt"]  
["checker.verify.dest.read.iops"]  
["checker.verify.dest.rps"]  
["checker.verify.dest.rt"]  
["checker.verify.source.read.iops"]  
["checker.verify.source.rps"]  
["checker.verify.source.rt"]  
["jdbcwriter.delay"]  
["jdbcwriter.iops"]  
["jdbcwriter.rps"]  
["store.conn"]  
["store.delay"]  
["store.iops"]  
["store.rps"]

# io.dropwizard.metrics5

## jetty 进程

OMS使用 io.dropwizard.metrics5 做部分监控, 外接口为: servlet形式

检查通路目录: /u01/ds/cm/package/deployapp/work/jetty-0.0.0.0-8088-cm.war-_-any-/webapp/WEB-INF/web.xml

```
        <servlet>
                <servlet-name>metrics</servlet-name>
                <servlet-class>com.codahale.metrics.servlets.AdminServlet</servlet-class>
        </servlet>
        <servlet-mapping>
                <servlet-name>metrics</servlet-name>
                <url-pattern>/metrics/*</url-pattern>
        </servlet-mapping>
``` 

打开url: <http://10.186.62.73:8088/metrics>

![image2022-12-1 19:1:16.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-1%2019%3A1%3A16.png)

可以直接获取堆栈: 

![image2022-12-1 19:2:6.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-1%2019%3A2%3A6.png)

也可以获取监控项 (通路进程): 

com.alibaba.drc.service.impl.ClientServiceImpl.batch-commit-size  
com.alibaba.drc.service.impl.ClientServiceImpl.batch-register-size  
com.alibaba.drc.service.impl.ClientServiceImpl.batch-commit  
com.alibaba.drc.service.impl.ClientServiceImpl.batch-register  
com.alibaba.drc.service.impl.ClientServiceImpl.get-client-id  
com.alibaba.drc.service.impl.ClientServiceImpl.get-client-partition  
com.alibaba.drc.service.impl.ClientServiceImpl.get-partition-offset  
com.alibaba.drc.service.impl.ClientServiceImpl.register-and-commit-total  
com.alibaba.drc.service.impl.DataBaseService.get-partition-offset

## coordinator 进程

由 DefaultCoordinator.initTimedTask 初始化 ScheduledLogReporterTask 任务, 每10s将metrics中的监控值打印到日志中

日志路径: 

/u01/ds/run/10.186.62.73-9000:p_48wfv693y1b4_dest-000-0:0000000001/logs/msg/metrics.log

样例: 

```
[2022-12-03 00:00:01.085] [{"jvm":{"JVM":"jvm:[heapMemory[max:3943MB, init:252MB, used:39MB, committed:243MB], noHeapMemory[max:0MB, init:2MB, used:47MB, committed:49MB], gc[gcName:ParNew, count:258, time:2054ms;gcName:ConcurrentMarkSweep, count:1, time:28ms;], thread[count:41]]"},"sink":{"sink_tps":0.0,"sink_worker_num":0,"sink_rps":0.0,"sink_total_transaction":5.0,"sink_total_record":5.0,"sink_commit_time":0.0,"sink_iops":0.0,"sink_worker_num_all":16,"sink_execute_time":0.0,"sink_total_bytes":2515.0},"source":{"source_record_num":15.0,"StoreSource[0]source_delay":8,"source_iops":214.98,"source_rps":0.0,"StoreSource[0]source_status":"running","source_dml_rps":0.0,"source_dml_num":5.0},"dispatcher":{"coordinator_dispatch_record_size":0,"dispatcher_ready_transaction_size":0},"frame":{"SourceTaskManager.createdSourceSize":1,"queue_slot1.batchAccumulate":0,"forward_slot0.batchAccumulate":0,"forward_slot0.batchCount":982063.0,"queue_slot1.batchCount":982063.0,"queue_slot1.rps":0.9998999834060669,"SourceTaskManager.sourceTaskNum":0,"forward_slot0.recordAccumulate":0,"forward_slot0.rps":0.9998999834060669,"queue_slot1.tps":0.9998999834060669,"forward_slot0.tps":0.9998999834060669,"queue_slot1.recordAccumulate":0,"queue_slot1.recordCount":982063.0,"forward_slot0.recordCount":982063.0}}]

[2022-12-03 00:00:11.084] [{"jvm":{"JVM":"jvm:[heapMemory[max:3943MB, init:252MB, used:39MB, committed:243MB], noHeapMemory[max:0MB, init:2MB, used:47MB, committed:49MB], gc[gcName:ParNew, count:258, time:2054ms;gcName:ConcurrentMarkSweep, count:1, time:28ms;], thread[count:41]]"},"sink":{"sink_tps":0.0,"sink_worker_num":0,"sink_rps":0.0,"sink_total_transaction":5.0,"sink_total_record":5.0,"sink_commit_time":0.0,"sink_iops":0.0,"sink_worker_num_all":16,"sink_execute_time":0.0,"sink_total_bytes":2515.0},"source":{"source_record_num":15.0,"StoreSource[0]source_delay":8,"source_iops":215.02,"source_rps":0.0,"StoreSource[0]source_status":"running","source_dml_rps":0.0,"source_dml_num":5.0},"dispatcher":{"coordinator_dispatch_record_size":0,"dispatcher_ready_transaction_size":0},"frame":{"SourceTaskManager.createdSourceSize":1,"queue_slot1.batchAccumulate":0,"forward_slot0.batchAccumulate":0,"forward_slot0.batchCount":982073.0,"queue_slot1.batchCount":982073.0,"queue_slot1.rps":1.000100016593933,"SourceTaskManager.sourceTaskNum":0,"forward_slot0.recordAccumulate":0,"forward_slot0.rps":1.000100016593933,"queue_slot1.tps":1.000100016593933,"forward_slot0.tps":1.000100016593933,"queue_slot1.recordAccumulate":0,"queue_slot1.recordCount":982073.0,"forward_slot0.recordCount":982073.0}}]
``` 

监控项: 

  - sink
    - sink_tps: Sink阶段, 每秒回放完成多少事务 (在事务回放完成时统计, releaseTransaction)

    - sink_total_transaction: Sink阶段, 回放事务的总数 (releaseTransaction)

    - sink_worker_num: Sink阶段, 工作线程中忙碌线程的数量

    - sink_worker_num_all: Sink阶段, 工作线程的总数

    - sink_rps: Sink阶段, 每秒回放完成多少行记录 (releaseTransaction)

    - sink_total_record: Sink阶段, 回放的行记录总数 (releaseTransaction)

    - sink_execute_time: Sink阶段, 行记录的平均执行时间 = 事务的SQL执行时间 / 事务的记录数 (releaseTransaction)

    - sink_commit_time: Sink阶段, 事务的提交阶段的平均时间 (releaseTransaction)

    - sink_iops: Sink阶段, 每秒回放的事务的行记录大小, 单位为kb/s (releaseTransaction)

    - sink_total_bytes: Sink阶段, 回放的事务的行记录大小总数

  - source
    - source_record_num: Source阶段, 处理的行记录的总数 (在处理行记录(将行记录拼接为事务)时统计, TransactionAssembler.notify)

    - source_rps: Source阶段, 每秒处理的行记录数 (TransactionAssembler.notify)

    - source_iops: Source阶段, 每秒处理的行记录的大小 (TransactionAssembler.notify)

    - source_dml_num: Source阶段, 处理的DML的行记录的总数 (TransactionAssembler.notify)

    - source_dml_rps: Source阶段, 每秒处理的DML的行记录数 (TransactionAssembler.notify)

    - StoreSource[0]source_status: Source阶段的状态, running/stopped

    - StoreSource[0]source_delay: Source阶段的延迟, 接收到数据的时间 - 原数据的时间戳 (TODO: 何时生成?) (TransactionAssembler.notify)

    - TODO: 多个source通道, 如何区分ID

  - dispatcher
    - coordinator_dispatch_record_size: Source阶段已完成, 但下游还没有消费, 积压在Source阶段的记录数 (改名为 wait_dispatch_record_size)
    - dispatcher_ready_transaction_size: Source阶段已完成, 但下游还没有消费, 积压在Source阶段的事务数 (改名为 ready_execute_batch_size)
  - frame
    - SourceTaskManager.createdSourceSize: SourceTaskManager启动的FORWARD的线程数

    - SourceTaskManager.sourceTaskNum: SourceTaskManager启动的非FORWAR的线程数

    - queue_slot1.batchAccumulate: 当前队列中的积压的事务数

    - queue_slot1.batchCount: 队列处理的事务总数

    - queue_slot1.tps: 队列处理的事务速率

    - queue_slot1.rps: 队列处理的行记录速率

    - queue_slot1.recordAccumulate: 当前队列中的积压的行记录数

    - queue_slot1.recordCount: 队列处理的行记录的总数

    - forward_slot0.batchAccumulate

    - forward_slot0.batchCount

    - forward_slot0.recordAccumulate

    - forward_slot0.recordCount

    - forward_slot0.rps

    - forward_slot0.tps

    - forward和queue slot的任务链
      - store进程 -> (source) forward任务 -> queue任务 -> TransactionScheduler -> sink

# 举例分析

DISPATCHER  
---  
wait_dispatch_record_size| ![image2022-12-26 13:36:41.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A36%3A41.png)  
ready_execute_batch_size| ![image2022-12-26 13:36:49.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A36%3A49.png)  
SOURCE  
source_status| running  
source_rps| ![image2022-12-26 13:36:58.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A36%3A58.png)  
source_delay| ![image2022-12-26 13:37:5.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A37%3A5.png)  
source_dml_rps| ![image2022-12-26 13:37:14.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A37%3A14.png)  
source_iops| ![image2022-12-26 13:37:23.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A37%3A23.png)  
source_dml_num| ![image2022-12-26 13:37:31.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A37%3A31.png)  
source_record_num| ![image2022-12-26 13:38:21.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A38%3A21.png)  
SINK  
sink_worker_number| ![image2022-12-26 13:38:29.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A38%3A29.png)  
sink_rps| ![image2022-12-26 13:38:38.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A38%3A38.png)  
sink_total_transaction| ![image2022-12-26 13:38:47.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A38%3A47.png)  
sink_iops| ![image2022-12-26 13:38:55.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A38%3A55.png)  
sink_tps| ![image2022-12-26 13:39:3.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A39%3A3.png)  
sink_commit_time| ![image2022-12-26 13:39:10.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A39%3A10.png)  
sink_total_record| ![image2022-12-26 13:39:18.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A39%3A18.png)  
sink_total_types| ![image2022-12-26 13:39:26.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A39%3A26.png)  
sink_execute_time| ![image2022-12-26 13:39:34.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A39%3A34.png)  
  
## 问题1

source_dml_rps和source_rps的差异, DML包括 INSERT/UPDATE/DELETE, 差异部分是? 

![image](http://8.134.54.170:8330/download/attachments/2130278/image2022-12-26%2013%3A37%3A14.png?version=1&modificationDate=1672033034983&api=v2)

![image](http://8.134.54.170:8330/download/attachments/2130278/image2022-12-26%2013%3A36%3A58.png?version=1&modificationDate=1672033018389&api=v2)

## 指标分析 source_delay

source_delay 是 coordinator 接收到行记录的时间 - 行记录的原始时间戳. source_delay不断上升, 意味着store进程产生了更多积压

  

## 问题2

store进程是否会产生无限积压, 消耗大量磁盘

  

## 指标分析 sink_rps

![image2022-12-26 13:52:24.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A52%3A24.png)

![image2022-12-26 13:51:40.png](/assets/01KJBYXQCY99CVFVFTJ7M39MJD/image2022-12-26%2013%3A51%3A40.png)

  

sink_rps 与 source_dml_rps 的差, 是整个coordinator处理的效率指标

  

## 指标分析问题

什么指标 标志着OB入库性能差?

目前的逻辑是 store产生了积压, coordinator没有效率差, 推测OB入库性能差
