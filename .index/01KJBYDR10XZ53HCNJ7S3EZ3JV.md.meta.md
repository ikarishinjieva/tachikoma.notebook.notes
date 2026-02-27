---
note: 01KJBYDR10XZ53HCNJ7S3EZ3JV.md
title: 20210721 - p_s 内存统计, 线程统计值和全局统计值的关系
indexed_at: 2026-02-27T10:07:10.052077+00:00
---

用户要求我分析一篇关于 MySQL performance_schema 内存统计的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：MySQL 8.0.25 的 performance_schema 内存监控，特别是关于临时表内存分配的全局统计值和线程统计值的关系

2. 核心内容：
- 开启 p_s 内存监控
- 对表进行自我翻倍操作
- 观察全局和局部（线程级）内存状况
- 通过 GDB 调试发现内存分配的调用栈

3. 关键概念：
- performance_schema 内存监控
- 全局内存统计 vs 线程内存统计
- temptable 临时表内存分配
- physical_disk 内存事件

4. 关联笔记分析：
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 → 直接相关，都是关于 performance_schema
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 → 相关，都是关于内存分析
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表，INFORMATION_SCHEMA.FILES 和 performance_schema.file_summary_by_instance 观测不一致 → 高度相关，都涉及临时表和 performance_schema 观测

现在生成索引元数据。
## 标签
MySQL, performance_schema, 内存统计, 临时表, GDB 调试, 线程统计

## 摘要
记录 MySQL 8.0.25 中 performance_schema 内存监控的实验测试，通过 GDB 调试分析临时表内存分配时全局统计值与线程统计值的关系。揭示了 temptable 模块中 physical_disk 内存事件的分配调用链路。

## 关键概念
- performance_schema 内存监控: MySQL 用于追踪内存分配事件的内置性能监控组件
- 全局/线程内存统计: memory_summary_global_by_event_name 与 memory_summary_by_thread_by_event_name 两种观测维度
- temptable 临时表: MySQL 使用 MMAP_FILE 等方式在磁盘上分配临时存储的表
- PFS_memory_safe_stat: performance_schema 中负责统计内存分配计数的核心代码模块

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同属 performance_schema 使用主题，该笔记记录日常使用方法
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同为 MySQL 内存分析主题，探索内存去向问题
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 同样涉及临时磁盘表与 performance_schema 观测不一致的问题
