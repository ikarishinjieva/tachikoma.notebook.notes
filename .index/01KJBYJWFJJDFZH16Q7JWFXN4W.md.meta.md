---
note: 01KJBYJWFJJDFZH16Q7JWFXN4W.md
title: 20220627 - p_s 中 io 统计次数的来源
indexed_at: 2026-03-05T08:03:18.867411+00:00
---

## 摘要
追踪 MySQL 8.0 中 io_global_by_file_by_latency 视图的数据来源，从 sys_schema 视图逐层定位到 performance_schema.file_summary_by_instance 表及底层代码实现。使用 GDB 监控 pfs_end_file_wait_vc 函数与 syscall 调用，分析 P_S 统计与实际系统调用的对应关系。

## 关键概念
- performance_schema.file_summary_by_instance: 存储文件 IO 统计信息的性能模式表
- pfs_end_file_wait_vc: Performance Schema 中记录文件等待事件的底层函数
- PSI_file_locker: MySQL 文件 instrumentation 接口，用于追踪文件操作
- inline_mysql_file_write: 封装 my_write 的函数，触发 P_S 文件等待统计

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 介绍 performance_schema 的日常使用方法和监控表
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 讨论 file_summary_by_instance 表与 INFORMATION_SCHEMA.FILES 观测不一致的问题
