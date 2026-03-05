---
note: 01KJBYD9T7VWC3J9T1JMVWGWMB.md
title: 20210330 - performance_schema的日常使用
indexed_at: 2026-03-05T07:20:18.584331+00:00
---

## 摘要
记录 performance_schema 日常使用的三条核心配置命令，通过 sys 库快速启用监控工具。包括启用 instrument、启用 consumer、禁用后台线程的标准化配置流程。

## 关键概念
- sys.ps_setup_enable_instrument: 启用 performance_schema 的性能监控采集器
- sys.ps_setup_enable_consumer: 启用 performance_schema 的数据消费表
- sys.ps_setup_disable_background_threads: 禁用后台线程监控以减少开销

## 关联笔记
- 01KJBYDR10XZ53HCNJ7S3EZ3JV.md: 同样使用 sys.ps_setup 系列命令配置 performance_schema 监控
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 使用 performance_schema.file_summary_by_instance 观测临时磁盘表
- 01KJBZB3ESK010EVRSR0XJDJFA.md: 使用 performance_schema.events_statements_summary_by_digest 分析 SQL 语句
