---
note: 01KJBYXQCY99CVFVFTJ7M39MJD.md
title: 20221201 - OMS监控
indexed_at: 2026-02-27T10:22:22.793998+00:00
---

Let me analyze this note about OMS monitoring to generate the index metadata.

The note is about OMS (likely OceanBase Migration Service or similar) monitoring, covering:
1. InfluxDB monitoring items
2. io.dropwizard.metrics5 for monitoring with jetty process and coordinator process
3. Various metrics and log paths

Let me create the metadata:

## 标签
监控，OMS, InfluxDB, metrics, JVM, 性能分析

## 摘要
本文记录 OMS 系统的监控体系，包括 InfluxDB 中的监控项和 io.dropwizard.metrics5 实现的监控接口。介绍了 jetty 进程的 metrics 访问方式和 coordinator 进程的监控日志输出。

## 关键概念
- InfluxDB: 时序数据库，存储 OMS 监控指标数据
- io.dropwizard.metrics5: Java 性能监控库，用于采集 JVM 和应用指标
- metrics servlet: 通过 HTTP 接口暴露监控数据的 servlet
- ScheduledLogReporterTask: 定时任务，每 10 秒将监控值打印到日志

## 关联笔记
Looking at the other notes, I need to find related ones:
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 (related to monitoring/observability)
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 (related to JVM memory monitoring)
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 (related to memory monitoring)


- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错 (JDK related, but not directly related to monitoring)

The most relevant connections are performance_schema for database monitoring, memory analysis notes for JVM insights, and memory metrics explanations.## 标签
监控，OMS, InfluxDB, metrics, JVM, 性能分析

## 摘要
本文记录 OMS 系统的监控体系，包括 InfluxDB 中的监控项和 io.dropwizard.metrics5 实现的监控接口。介绍了 jetty 进程的 metrics 访问方式和 coordinator 进程的监控日志输出。

## 关键概念
- InfluxDB: 时序数据库，存储 OMS 监控指标数据
- io.dropwizard.metrics5: Java 性能监控库，用于采集 JVM 和应用指标
- metrics servlet: 通过 HTTP 接口暴露监控数据的 servlet
- ScheduledLogReporterTask: 定时任务，每 10 秒将监控值打印到日志

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用（同为监控/可观测性相关主题）
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了（涉及 JVM 内存监控分析）
