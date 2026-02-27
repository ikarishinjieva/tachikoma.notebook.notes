---
note: 01KJBYXQMP1JBTG1M6CHNYWF8E.md
title: 20221210 - 光大CK 调大后台线程数导致内存急速增长
indexed_at: 2026-02-27T10:23:39.749469+00:00
---

用户要求我分析一篇关于 ClickHouse 调大后台线程数导致内存问题的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **核心问题**：调大 ClickHouse 的 background_pool_size（16->32）后，内存急速上涨
2. **现象**：多个线程在执行 mutate part 操作时消耗大量内存（约 187GiB）
3. **分析**：通过日志分析各线程的内存使用情况

**标签**应该包括：ClickHouse、内存问题、线程池、性能调优、故障排查

**摘要**：记录调大 ClickHouse 后台线程池后内存暴涨的问题排查过程。通过日志分析发现多个后台线程在执行 mutate 操作时各消耗约 20GiB 内存，总计约 187GiB。

**关键概念**：
- background_pool_size: ClickHouse 后台操作线程池大小
- mutate part: ClickHouse 对数据分片进行变异的后台操作
- ReplicatedMergeTreeQueue: ClickHouse 复制表的队列机制
- MemoryTracker: ClickHouse 内存追踪器

**关联笔记**：
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 20210501 - Clickhouse 集群 Distributed Engine 学习 - 都是 ClickHouse 相关
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习 - ClickHouse 相关


- 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究 - ClickHouse 相关
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+学习笔记 - ClickHouse 相关

这些笔记都涉及 ClickHouse 的不同方面，可以作为参考关联。

不过应该优先选择最相关的几篇。## 标签
ClickHouse, 内存问题, 线程池, 性能调优, 故障排查, background_pool_size

## 摘要
记录调大 ClickHouse 后台线程池（background_pool_size 从 16 增至 32）后内存急速上涨的问题排查过程。通过日志分析发现多个后台线程在执行 mutate part 操作时各消耗约 20GiB 内存，总计约 187GiB。

## 关键概念
- background_pool_size: ClickHouse 后台操作线程池大小配置参数
- mutate part: ClickHouse 对数据分片进行变异合并的后台操作
- ReplicatedMergeTreeQueue: ClickHouse 复制表引擎的队列机制，负责加载 mutation 任务
- MemoryTracker: ClickHouse 内存追踪器，用于监控各线程内存用量

## 关联笔记
- 01KJBYDD58AJTW5NKT3GET7B89.md: 同为 ClickHouse 学习笔记，内容相关
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: ClickHouse 集群 Distributed Engine 学习，涉及 ClickHouse 架构
- 01KJBYDBWTJGYAJWBZ93943E5R.md: ClickHouse Join 研究，同为 ClickHouse 技术分析笔记
