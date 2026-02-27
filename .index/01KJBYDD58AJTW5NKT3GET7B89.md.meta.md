---
note: 01KJBYDD58AJTW5NKT3GET7B89.md
title: 20221022 - 旧: Clickhouse+学习笔记
indexed_at: 2026-02-27T10:04:29.955911+00:00
---

用户希望我分析这篇 Clickhouse 学习笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20221022 - 旧：Clickhouse+学习笔记
2. 内容包含：
   - 目录结构
   - 业务场景/数据集（各种 benchmark 和数据集链接）
   - 文档（Clickhouse 稀疏索引、Merge tree 等）
   - 执行计划、异常现象、知识框架等

3. 其他笔记列表中，有几篇与 Clickhouse 相关：
   - 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 20210501 - Clickhouse 集群 Distributed Engine 学习
   - 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习
   - 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究
   - 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 20210622 - Clickhouse distributed join 学习

这些笔记都与 Clickhouse 相关，可以作为关联笔记。

现在生成元数据：

标签：ClickHouse, 数据库, 学习笔记, 索引, 性能优化, 数据集

摘要：这篇笔记记录了 ClickHouse 数据库的核心概念和学习资源，包括业务场景、数据集、文档资料等。内容涵盖稀疏索引、MergeTree 引擎、执行计划分析等关键技术点。


关键概念包括稀疏索引（默认 8192 行粒度，非唯一索引）、MergeTree（ClickHouse 核心存储引擎，支持后台 Merge 操作）、Distributed Table（逻辑视图，数据存储在 local table）以及 MaterializedMySQL（与 MySQL 集成的物化表）。

关联笔记中，01KJBYDAYBCJZXBGJFXXQ48F5X.md 涉及 MaterializedMySQL 相关内容。

01KJBYDAQ6YMA4RN9VCSAXHRZ0.md 和 01KJBYDBWTJGYAJWBZ93943E5R.md 分别补充了 Distributed Engine 和 Join 机制的学习内容。## 标签
ClickHouse, 数据库, 学习笔记, 稀疏索引, MergeTree, 性能优化

## 摘要
这篇笔记记录了 ClickHouse 数据库的核心学习内容和参考资料，包括业务场景、数据集、文档资料等。内容涵盖稀疏索引机制、MergeTree 引擎工作原理、执行计划分析等关键技术点。

## 关键概念
- 稀疏索引：默认粒度为 8192 行作为一个块，非唯一索引，point select 性能差
- MergeTree：ClickHouse 核心存储引擎，Merge 操作在后台进行
- Distributed Table：逻辑视图，实际数据存储在 local table
- MaterializedMySQL：与 MySQL 集成的物化表功能

## 关联笔记
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 同为主题 Clickhouse MaterializedMySQL 学习
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 同为主题 Clickhouse 集群 Distributed Engine 学习
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 同为主题 Clickhouse Join 研究
