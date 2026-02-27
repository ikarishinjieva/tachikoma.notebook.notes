---
note: 01KJBYEK8DZVDASX1CPQPXC5DA.md
title: 20210830 - THP的作用探索
indexed_at: 2026-02-27T10:10:06.704510+00:00
---

用户要求我分析一篇关于 THP（Transparent Huge Pages，透明大页）的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主要讲述如何观察 THP 对 TLB 的作用
2. 使用了两组测试：C 代码测试和 sysbench 测试
3. 对比了 THP enabled 和 disabled 两种情况下的性能差异
4. 涉及的技术概念：THP、TLB、page walk、perf 性能分析等

现在我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

关于关联笔记，我需要从提供的列表中找相关的：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 可能与内存相关
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 内存相关
- 01KJBYDSKSCBFSFV4QPZYZ5RX.md: 20210407 - 同样的压力，不同机器 perf 热点不同 - 都使用了 perf 分析

这些是比较相关的。
## 标签
THP, TLB, 性能优化, 内存管理, perf, page walk

## 摘要
通过 C 代码和 sysbench 测试对比 THP 开启与关闭场景下的 TLB miss 指标。实验证明开启 THP 可显著减少 page walk 开销，执行时间从 2.45s 降至 1.65s，性能提升约 33%。

## 关键概念
- THP (Transparent Huge Page): Linux 透明大页技术，自动合并小页为大页减少 TLB miss
- TLB (Translation Lookaside Buffer): 页表缓存，加速虚拟地址到物理地址的转换
- Page Walk: TLB miss 时遍历页表的过程，开销较大
- dTLB/iTLB: 数据 TLB 和指令 TLB，分别处理数据和指令的地址转换

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 同为内存性能分析主题
- 01KJBYDSKSCBFSFV4QPZYZ5RX.md: 20210407 - 同样的压力，不同机器 perf 热点不同 - 都使用 perf 进行性能分析
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 同为内存相关技术分析
