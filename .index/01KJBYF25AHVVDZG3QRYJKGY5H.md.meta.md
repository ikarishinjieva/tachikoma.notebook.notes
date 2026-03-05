---
note: 01KJBYF25AHVVDZG3QRYJKGY5H.md
title: 20211009 - 监控MySQL内存的命令
indexed_at: 2026-03-05T07:47:05.383657+00:00
---

## 摘要
介绍使用 tcmalloc 和 jemalloc 监控 MySQL 内存分配的两种方案。仅用 tcmalloc 可监控 mmap，混合使用可同时监控 mmap 和 malloc，但存在数据偏差和 pprof 报错问题。

## 关键概念
- HEAPPROFILE: gperftools 的堆内存分析工具，生成内存分配快照文件
- mmap/malloc: 两种内存分配方式，mmap 用于大块内存，malloc 用于常规分配
- LD_PRELOAD: Linux 环境变量，用于在程序加载前预加载共享库
- pprof: 性能分析工具，用于分析和可视化 heap.prof 文件

## 关联笔记
- 01KJBYF1Q3J13H7W6JV7FNVXN3.md: 讲解如何用 GDB 调试 jemalloc 和 tcmalloc 的方法
- 01KJBYF0S1KGHWDAD4XA57BJNG.md: 对比 jemalloc 和 tcmalloc 的内存统计行为差异
- 01KJBYEX76SJWVW62660M31PCG.md: 解释 tcmalloc 统计 MySQL 内存分配的流程
