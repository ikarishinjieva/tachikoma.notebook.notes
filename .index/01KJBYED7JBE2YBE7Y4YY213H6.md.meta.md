---
note: 01KJBYED7JBE2YBE7Y4YY213H6.md
title: 20210819 - MySQL 内存, 除了p_s统计的, 还有哪些 3 - tcmalloc
indexed_at: 2026-03-05T07:41:12.551791+00:00
---

## 标签
MySQL, 内存分析, tcmalloc, redo log, double write buffer, 性能分析

## 摘要
使用 tcmalloc 分析 MySQL 8.0.25 内存分配，识别主要内存消耗点包括 redo log 缓冲、Link_buf 并发结构、double write buffer 和文件系统块缓存。对比 os 统计 (RSS 426MB) 与 tcmalloc 统计 (330MB) 差异，通过 smaps 分析堆内存和代码段分布。

## 关键概念
- tcmalloc: Google 性能分析工具，用于追踪 malloc/new 内存分配
- Link_buf: InnoDB 用于 redo log 并发写入和脏页刷新的链表缓冲结构
- double write buffer: InnoDB 页写入机制，防止部分页写入导致的数据损坏
- 块缓存 (Block Cache): 文件系统层缓存，用于引擎加解密操作，默认 128 个块

## 关联笔记
- 01KJBYECA7QEGNNAHJQ0PZ8QPE.md: 同一系列第 2 篇，MySQL 内存分析续篇
- 01KJBYDV2NWRKK7EW365CFMZG6.md: 同一系列第 1 篇，MySQL 内存分析起始篇
- 01KJBYEFDGHWM1MZ83G5VKK9QS.md: double write buffer 详细机制说明
