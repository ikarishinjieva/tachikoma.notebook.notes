---
note: 01KJBYECA7QEGNNAHJQ0PZ8QPE.md
title: 20210819 - MySQL 内存, 除了p_s统计的, 还有哪些 2
indexed_at: 2026-03-05T07:40:52.908513+00:00
---

## 标签
MySQL, 内存分析, bcc, memleak, performance_schema, 内存追踪

## 摘要
介绍使用 bcc 工具改造 memleak 脚本，通过追踪 malloc 与 pfs_memory_alloc_vc 并相互抵消，识别未被 MySQL performance_schema 统计的内存分配。提供了完整的 Python 脚本实现和使用方法。

## 关键概念
- bcc: Linux 内核追踪工具框架，用于编写 eBPF 程序监控系统调用和内核函数
- memleak: bcc 自带的内存泄露追踪脚本，可追踪 malloc/free 调用
- pfs_memory_alloc_vc: MySQL performance_schema 的内存分配统计入口函数

## 关联笔记
- 01KJBYDV2NWRKK7EW365CFMZG6.md: 前篇笔记，介绍使用 gdb 追踪 malloc 和 pfs_memory_alloc_vc 的初始方法
