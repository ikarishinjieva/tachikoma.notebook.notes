---
note: 01KJBYDR10XZ53HCNJ7S3EZ3JV.md
title: 20210721 - p_s 内存统计, 线程统计值和全局统计值的关系
indexed_at: 2026-03-05T07:37:04.644504+00:00
---

## 摘要
通过 performance schema 监控 MySQL 8.0.25 内存分配，使用 sys.ps_setup 开启 memory 事件监控。结合 gdb 调试分析 temptable 引擎的内存分配调用链，揭示全局统计与线程级统计值的对应关系。

## 关键概念
- performance_schema: MySQL 性能监控组件，收集服务器执行时的底层事件信息
- memory_summary_global_by_event_name: 全局内存统计视图，按事件名称汇总所有内存分配
- memory_summary_by_thread_by_event_name: 线程级内存统计视图，按线程和事件名称分类
- temptable: MySQL 8.0 默认内部临时表存储引擎，支持内存和 mmap 文件两种分配方式
- PFS_memory_safe_stat: Performance Schema 中负责安全计数内存分配的类

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同属 performance_schema 配置与日常使用主题
- 01KJBZA51RVTP0C0VXYHQ96C2D.md: 涉及 MySQL8.0 temptable 引擎导致内部临时表无法转磁盘的 bug
- 01KJBZB3ESK010EVRSR0XJDJFA.md: 详细讨论 MySQL 内部临时表的使用场景和机制
