---
note: 01KJBYXQWQNYSHP23Q9B3MDJCR.md
title: 20221020 - 光大clickhouse Too many parts报错
indexed_at: 2026-02-27T10:24:52.112478+00:00
---

用户要求我分析一篇关于 ClickHouse 报错的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 标题：20221020 - 光大 clickhouse Too many parts 报错
2. 内容主要是关于 ClickHouse 的 AST is too big 报错分析，涉及复制表（ReplicatedMergeTree）的延迟问题

我需要根据笔记内容生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从提供的列表中选择相关的）

关联笔记分析：
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 20210501 - Clickhouse 集群 Distributed Engine 学习 - 相关，都是 ClickHouse
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习 - 相关，都是 ClickHouse
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究 - 相关，都是 ClickHouse
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+ 学习笔记 - 相关，都是 ClickHouse，时间接近
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 20210622 - Clickhouse distributed join 学习 - 相关，都是 ClickHouse


- 最相关的是 01KJBYDD58AJTW5NKT3GET7B89.md，因为时间最接近（20221022 vs 20221020），且都是 ClickHouse 学习笔记

现在生成索引元数据。
## 标签
ClickHouse, AST is too big, ReplicatedMergeTree, 复制表, 错误分析, 性能问题

## 摘要
笔记记录了光大银行 ClickHouse 集群中因 AST 过大导致的复制表同步延迟问题。通过分析 27 节点日志，定位到 DB::StorageReplicatedMergeTree::queueTask() 在处理复制队列时触发 AST size 限制（500000），阻塞了部分合并/变异操作。

## 关键概念
- AST is too big: ClickHouse 对抽象语法树大小的限制，默认最大 500000 节点
- StorageReplicatedMergeTree: ClickHouse 的复制表引擎，通过 ZooKeeper 协调多副本同步
- queueTask: 复制表后台任务处理函数，负责执行复制队列中的日志条目
- Mutation: ClickHouse 的表数据变更操作（如 ALTER UPDATE/DELETE），会触发全部分区重写

## 关联笔记
- 01KJBYDD58AJTW5NKT3GET7B89.md: 同为 ClickHouse 学习笔记，时间相近（20221022）
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: ClickHouse 集群 Distributed Engine 学习，涉及集群架构
- 01KJBYDBWTJGYAJWBZ93943E5R.md: ClickHouse Join 研究，同为 ClickHouse 技术问题
