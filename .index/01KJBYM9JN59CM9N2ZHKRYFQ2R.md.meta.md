---
note: 01KJBYM9JN59CM9N2ZHKRYFQ2R.md
title: 20220819 - 如何理解perf event: node-load-misses
indexed_at: 2026-02-27T10:18:18.066369+00:00
---

用户要求我分析一篇关于 perf event 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20220819 - 如何理解 perf event: node-load-misses
2. 内容主要关于：
   - 查看 Linux 源码中 Skylake 的 hardware cache event 定义
   - node-load 的标志是 L3_MISS_LOCAL_DRAM
   - node-load-misses 的标志是 L3_MISS_REMOTE
   - 一个是本地读，一个是远程读

标签应该包括：perf, performance analysis, CPU cache, NUMA, Linux kernel 等

摘要：这篇笔记分析了 perf event 中 node-load-misses 的含义，通过查看 Linux 源码中 Skylake 的 hardware cache event 定义，解释了 node-load 和 node-load-misses 分别对应本地内存读取和远程内存读取。

关键概念：
- perf event: Linux 性能分析工具的事件机制
- node-load: L3 缓存未命中时从本地 DRAM 读取
- node-load-misses: L3 缓存未命中时从远程节点读取
- L3_MISS: L3 缓存未命中标志

关联笔记：
查看其他笔记列表，寻找与 perf、性能分析相关的：
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 20210407 - 同样的压力，不同机器 perf 呈现的热点不同 - 这个与 perf 相关


- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 与内存分析相关
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 与内存相关

最相关的是 perf 那篇。## 标签
perf, 性能分析, CPU 缓存, NUMA, Linux 内核, Skylake

## 摘要
这篇笔记通过分析 Linux 内核源码中 Skylake 的 hardware cache event 定义，解释了 perf event 中 node-load 和 node-load-misses 的含义。node-load 对应 L3 未命中时从本地 DRAM 读取，node-load-misses 对应从远程节点读取。

## 关键概念
- perf event: Linux 性能分析工具，用于采集硬件事件
- node-load: L3 缓存未命中时从本地内存节点读取数据
- node-load-misses: L3 缓存未命中时从远程内存节点读取数据
- L3_MISS: L3 缓存未命中的硬件事件标志

## 关联笔记
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 同样涉及 perf 性能分析，讨论不同机器上 perf 热点差异
