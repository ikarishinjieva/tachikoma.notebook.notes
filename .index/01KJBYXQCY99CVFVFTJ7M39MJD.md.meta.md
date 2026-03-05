---
note: 01KJBYXQCY99CVFVFTJ7M39MJD.md
title: 20221201 - OMS监控
indexed_at: 2026-03-05T08:21:29.635035+00:00
---

## 标签
OMS, 监控, InfluxDB, metrics, coordinator, 数据同步

## 摘要
记录 OMS 系统的监控体系，包括 InfluxDB 存储的监控项和 io.dropwizard.metrics5 实现的监控接口。涵盖 jetty 进程的/metrics 接口和 coordinator 进程的 metrics.log 日志输出。

## 关键概念
- InfluxDB: 存储 OMS 监控数据的时序数据库，包含 checker、jdbcwriter、store 等监控项
- io.dropwizard.metrics5: OMS 使用的监控框架，通过 servlet 形式暴露监控接口
- coordinator 进程: 通过 ScheduledLogReporterTask 任务每 10s 将监控值打印到 metrics.log
- metrics.log: OMS 监控日志文件，记录 JVM、sink、source、dispatcher、frame 等维度的监控数据
- jetty servlet: 通过/metrics/*URL 模式暴露 AdminServlet 接口获取监控数据和堆栈信息

## 关联笔记
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: 整理 OMS 全量复制链路，涉及 coordinator 配置和 metrics 相关参数
- 01KJBZ3FQT8FKX47BF6Q8C4X7J.md: 分析 OMS 日志系统，包含 metrics.log 日志路径和输出机制
- 01KJBZ45XPJ5T7FYJVGSDF5EKH.md: 整理 OMS 全量复制链路过程，涉及 coordinator 进程启动和配置
