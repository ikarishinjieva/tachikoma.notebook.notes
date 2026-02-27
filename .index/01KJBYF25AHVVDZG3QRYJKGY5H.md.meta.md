---
note: 01KJBYF25AHVVDZG3QRYJKGY5H.md
title: 20211009 - 监控MySQL内存的命令
indexed_at: 2026-02-27T10:12:18.859968+00:00
---

用户要求我分析一篇关于 MySQL 内存监控的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：监控 MySQL 内存的命令
2. 主要内容：
   - 使用 tcmalloc 只监控 mmap 的方法
   - 使用 jemalloc + tcmalloc 同时监控 mmap 和 malloc 的方法
   - 两种方法的优缺点和注意事项

我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

关联笔记分析：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 与内存分析相关，有关联
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 与内存相关，有关联
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - 与 MySQL 监控相关，有一定关联

让我生成简洁的索引元数据。## 标签
MySQL, 内存监控, tcmalloc, jemalloc, mmap, malloc

## 摘要
介绍使用 tcmalloc 和 jemalloc 监控 MySQL 内存分配的两种方案。第一种只监控 mmap，力度较粗；第二种混合使用可分别得到 mmap 和 malloc 的结果，但需处理 pprof 报错问题。

## 关键概念
- HEAPPROFILE: gperftools 的堆内存分析工具，用于生成内存分配 profile
- tcmalloc: Google 开发的线程缓存内存分配器，支持内存监控
- jemalloc: BSD 授权的通用内存分配器，可与 tcmalloc 配合使用
- LD_PRELOAD: Linux 环境变量，用于在程序加载前预加载共享库

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同为内存分析主题，探讨内存分配去向
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 涉及内存指标解释，与内存监控相关
