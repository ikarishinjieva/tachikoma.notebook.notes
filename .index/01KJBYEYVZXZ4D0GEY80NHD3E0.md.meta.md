---
note: 01KJBYEYVZXZ4D0GEY80NHD3E0.md
title: 20210928 - tcmalloc 不同so包性能有差异
indexed_at: 2026-02-27T10:11:26.431896+00:00
---

用户要求我分析一篇关于 tcmalloc 性能差异的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 标题：20210928 - tcmalloc 不同 so 包性能有差异
2. 内容主要讲述：
   - 现象：libtcmalloc_minimal.so 与 libtcmalloc.so 性能差异很大
   - 使用 gperftools 2.9.1 版本
   - 在 MySQL 上进行测试，sysbench 压测
   - libtcmalloc_minimal: tps 10624.81 起步
   - libtcmalloc: tps 5.32 起步
   - 探索了编译日志的差异

现在我需要生成：
1. 标签（3-6 个）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

关于关联笔记，我需要看是否有相关的：
- 内存相关的：20210519 - 内存去哪儿了 (01KJBYDAF12RN18BECZG1QSTEC.md) - 可能有关联
- 性能相关的：20210407 - 同样的压力，不同机器 perf 呈现的热点不同 (01KJBYD9SKSCBFSFV4QPZYZ5RX.md) - 可能有关联
- 20210513 - 百胜 cpu 高的探索 (01KJBYDB30D5DK9MDJ70R5YFST.md) - 性能探索相关


- 其他笔记主要是 MySQL、Clickhouse 相关，与 tcmalloc 性能问题关联不大

这篇笔记的核心是记录 gperftools 两个库在 MySQL 场景下的性能对比，以及通过编译日志分析差异原因。## 标签
tcmalloc, gperftools, 性能优化, MySQL, 内存分配, 编译差异

## 摘要
记录 libtcmalloc_minimal.so 与 libtcmalloc.so 在 MySQL 场景下性能差异巨大的问题（tps 10624 vs 5.32）。通过对比编译日志分析两个库的编译参数和链接差异，定位性能差距原因。

## 关键概念
- libtcmalloc_minimal: tcmalloc 的精简版本，只包含基本内存分配功能
- libtcmalloc: tcmalloc 完整版本，包含堆检查、线程管理等额外功能
- HEAPPROFILE: gperftools 的堆内存分析工具环境变量
- sysbench: 数据库基准测试工具，用于压测 MySQL 性能

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同属内存分析主题）
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 20210407 - 同样的压力，不同机器 perf 呈现的热点不同（性能差异分析相关）
