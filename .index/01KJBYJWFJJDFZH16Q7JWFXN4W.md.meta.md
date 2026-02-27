---
note: 01KJBYJWFJJDFZH16Q7JWFXN4W.md
title: 20220627 - p_s 中 io 统计次数的来源
indexed_at: 2026-02-27T10:17:19.526988+00:00
---

This is a note about MySQL performance_schema IO statistics. Let me analyze it:

1. The note traces the source of IO statistics in performance_schema
2. It starts from io_global_by_file_by_latency view
3. Traces through the code to find where write counts come from
4. Uses GDB to trace the actual function calls
5. Has TODO items about finding P_S triggers without syscall

Key concepts:
- performance_schema (P_S)
- file_summary_by_instance
- pfs_end_file_wait_vc
- PSI_file interface
- IO statistics

Related notes:
- 01KJBYD9T7VWC3J9T1JMVGWMB.md: 20210330 - performance_schema 的日常使用 (directly related to performance_schema)
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表，INFORMATION_SCHEMA.FILES 和 performance_schema.file_summary_by_instance 观测不一致 (mentions file_summary_by_instance)
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 (related to IO/disk flush)

Let me create the index metadata.## 标签
MySQL, performance_schema, IO 统计，源码分析，GDB 调试

## 摘要
追踪 MySQL performance_schema 中 IO 统计次数的代码来源，从视图 io_global_by_file_by_latency 追溯到底层实现函数 pfs_end_file_wait_vc。通过 GDB 监控 mysqld 进程，分析 P_S 触发与 syscall 的关系，但尚未找到触发 P_S 却未触发 syscall 的堆栈。

## 关键概念
- performance_schema: MySQL 性能监控接口，提供运行时统计信息
- file_summary_by_instance: 按文件实例汇总的 IO 统计表
- pfs_end_file_wait_vc: 文件 IO 等待结束时的统计记录函数
- PSI_file_interface: MySQL 文件 instrumentation 接口
- inline_mysql_file_write: 封装实际写操作并触发性能统计的内联函数

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVGWMB.md: 同为主题为 performance_schema 的使用与研究
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 同样涉及 file_summary_by_instance 表的观测问题
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同为主题为 MySQL 刷盘与 IO 相关的研究
