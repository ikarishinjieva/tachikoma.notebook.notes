---
note: 01KJBYDD58AJTW5NKT3GET7B89.md
title: 20221022 - 旧: Clickhouse+学习笔记
indexed_at: 2026-03-05T07:29:59.162211+00:00
---

## 摘要
ClickHouse 综合学习笔记，涵盖业务场景、数据集、执行计划、知识框架等核心内容。重点记录稀疏索引机制、MergeTree 数据更新流程、Distributed table 架构及 MaterializedMySQL 等关键特性。

## 关键概念
- 稀疏索引: 默认粒度 8192 行为一个块，非唯一索引，point select 性能差
- MergeTree: ClickHouse 核心存储引擎，Merge 操作在后台进行
- Distributed table: 逻辑视图，实际数据存储在 local table
- ReplicatedMergeTree: 可实现异步主主复制
- MaterializedMySQL: MySQL 数据同步到 ClickHouse 的数据库引擎

## 关联笔记
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: MaterializedMySQL 专题学习，与本笔记 MaterializedMySQL 章节对应
- 01KJBYDBWTJGYAJWBZ93943E5R.md: ClickHouse Join 研究，与本笔记分布式表内容相关
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: Distributed Join 学习，与本笔记 Distributed table 内容相关
