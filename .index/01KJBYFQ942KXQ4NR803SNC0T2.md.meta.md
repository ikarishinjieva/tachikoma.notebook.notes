---
note: 01KJBYFQ942KXQ4NR803SNC0T2.md
title: 20211230 - 国债MySQL 8.0 insert性能与MySQL 5.7有差异
indexed_at: 2026-03-05T07:54:16.389123+00:00
---

## 标签
MySQL 性能分析，火焰图，InnoDB, perf 工具，page fault, 版本对比

## 摘要
通过火焰图、perf-stat、page fault 分析等手段，系统对比 MySQL 8.0 与 5.7 在批量 INSERT 场景下的性能差异。发现 8.0 的 log_buffer_write 调用次数比 5.7 多 20 倍，且存在大量 page fault，最终定位到内存分配和 redo log 写入机制的差异是性能下降的主因。

## 关键概念
- 火焰图 (Flame Graph): 可视化 CPU 时间或事件在函数调用栈上的分布，快速定位性能热点
- page fault: 进程访问未映射到物理内存的页面时触发的中断，频繁发生会影响性能
- innodb_checksum_algorithm: InnoDB 页校验和算法，设置为 none 可消除 CRC 计算开销
- off-CPU 火焰图: 记录线程等待状态（如 I/O、锁）的时间分布，用于分析阻塞原因
- log_buffer_write: InnoDB 将 redo 日志写入 log buffer 的函数，8.0 调用频率显著高于 5.7

## 关联笔记
- 01KJBYKK56GGEG65TSY8WHHZXJ.md: 同样使用 perf 工具诊断 MySQL 性能问题，涉及 context-switches 和 page-faults 分析
- 01KJBYF0S1KGHWDAD4XA57BJNG.md: 使用 perf 观测 MySQL 内存分配行为，涉及 mmap 和内存分配器对比
- 01KJBZEE9JTH4RKQG664GFW88H.md: MySQL 8.0 crash 问题排查，涉及 innodb 锁和 performance_schema 分析
