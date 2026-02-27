---
note: 01KJBYVTTVNYTSRBX5JBT2MJKP.md
title: 20221123 - 光大 AST is too big 报错后续
indexed_at: 2026-02-27T10:20:44.448671+00:00
---

用户要求我分析一篇关于 ClickHouse 数据库问题的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 标题：20221123 - 光大 AST is too big 报错后续
2. 内容主要关于：
   - ClickHouse 表参数变更未生效的问题
   - 诊断发现是后台线程池资源不足导致 mutation 无法执行
   - 给出了几个解决方案建议

我需要提取：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从提供的列表中找相关的）

看关联笔记列表，相关的可能是：
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 20210501 - Clickhouse 集群 Distributed Engine 学习
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20210622 - Clickhouse distributed join 学习
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+学习笔记

最直接相关的是 01KJBYDD58AJTW5NKT3GET7B89.md（20221022 - 旧：Clickhouse+学习笔记），因为笔记标题提到这是"光大 AST is too big 报错后续"，而前继笔记是"20221020 - 光大 clickhouse Too many parts 报错"，所以这篇应该是 ClickHouse 相关学习笔记的后续。


现在我来生成元数据。## 标签
ClickHouse, Mutation, 后台线程池, 参数调优, 故障诊断, ReplicatedMergeTree

## 摘要
记录 ClickHouse 表参数变更未生效问题的诊断过程，发现是后台线程池资源不足导致 mutation 无法执行。分析了 shouldExecuteLogEntry 返回 false 的原因，并给出扩大线程池、等待空闲或调整阈值参数等解决方案。

## 关键概念
- shouldExecuteLogEntry: 判断是否有足够资源执行 mutation 日志_entry_的函数
- background_pool_size: ClickHouse 后台线程池的总线程数
- number_of_free_entries_in_pool_to_execute_mutation: 执行 mutation 所需的最小空闲线程数阈值
- MUTATE_PART: ClickHouse 对数据分区执行变更操作的日志类型
- max_source_parts_size: mutation 执行时源分区的最大允许大小，为 0 表示资源不足

## 关联笔记
- 01KJBYDD58AJTW5NKT3GET7B89.md: 同为 ClickHouse 学习笔记，可能是同一系列问题记录
