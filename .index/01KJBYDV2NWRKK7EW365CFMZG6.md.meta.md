---
note: 01KJBYDV2NWRKK7EW365CFMZG6.md
title: 20210724 - MySQL 内存, 除了p_s统计的, 还有哪些
indexed_at: 2026-03-05T07:37:58.128254+00:00
---

## 摘要
介绍使用 gdb 在 malloc 和 pfs_memory_alloc_vc 两处设置断点，抽样捕获 MySQL 内存分配堆栈的观测方法。通过对比 OS 层 malloc 与 performance_schema 内存统计入口的调用栈，分析 MySQL 内存分配的实际路径。

## 关键概念
- pfs_memory_alloc_vc: performance_schema 的内存统计入口函数
- gdb 断点抽样: 使用 ignore 命令减少断点对性能的影响
- rw_lock: MySQL 的读写锁机制，在堆栈中频繁出现
- srv_master_thread: MySQL 后台主线程，负责空闲任务处理

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同样使用 gdb 分析 MySQL crash 的 coredump，涉及类似的堆栈分析方法
- 01KJBZAPGEQT0E9NB6C8CFG229.md: MySQL 问题排查方法论，与本文的观测方法形成互补
- 01KJBZAGQ948VMTJBEVJFDYEV6.md: MySQL 内核多步推理分析，涉及 performance_schema 的使用场景
