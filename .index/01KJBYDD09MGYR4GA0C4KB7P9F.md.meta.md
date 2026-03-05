---
note: 01KJBYDD09MGYR4GA0C4KB7P9F.md
title: 20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation
indexed_at: 2026-03-05T07:29:38.513340+00:00
---

## 摘要
探讨大规模分布式计算场景下的高级 Join 策略，包括 Shuffle Join、Broadcast Join 等优化技术。笔记附带学术论文 PDF，用于研究分布式 Join 的性能优化方案。

## 关键概念
- Broadcast Join: 将小表广播到所有节点避免 Shuffle 的 Join 策略
- Shuffle Join: 通过数据重分区实现大表关联的分布式 Join 方式
- Distributed Query: 跨多个计算节点协同执行的查询处理模式

## 关联笔记
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: ClickHouse distributed Join 学习，同主题后续研究笔记
- 01KJBYDBWTJGYAJWBZ93943E5R.md: ClickHouse Join 研究，涉及分布式 Join 的 Shuffle 机制分析
- 01KJBYZXHS7VYMQPF4FGZ6MMG1.md: MapReduceDocumentsChain 机制分析，涉及 MapReduce 分布式处理模式
