---
note: 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md
title: 20210501 - Clickhouse集群 Distributed Engine 学习
indexed_at: 2026-02-27T10:02:27.888075+00:00
---

用户要求我分析一篇关于 ClickHouse 集群 Distributed Engine 的笔记，并生成索引元数据。我需要：

1. 提取 3-6 个标签
2. 写 2-3 句话的摘要
3. 列出 3-5 个关键概念
4. 找出 0-3 篇关联笔记

从笔记内容来看，这是关于 ClickHouse 的 Distributed 表引擎的学习笔记，涵盖了：
- Distributed 引擎的工作原理
- 读写请求处理
- 负载均衡
- 分片配置
- 日志观察等

关联笔记方面，我看到有几篇 ClickHouse 相关的：
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 20210622 - Clickhouse distributed join 学习
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+学习笔记

其中 Clickhouse distributed join 学习与这篇笔记的 Distributed Engine 主题最相关。

让我生成索引元数据。
## 标签
ClickHouse, Distributed Engine, 分布式查询, 分片, 负载均衡, 集群

## 摘要
本文学习 ClickHouse 的 Distributed 表引擎，包括其不存储数据、仅作为逻辑表转发请求的特性。涵盖读写请求处理机制、负载均衡策略、分片配置及调试方法。

## 关键概念
- Distributed Engine: 不实际存储数据，仅作为逻辑表将请求分发到各分片节点
- internal_replication: 控制写请求是发往第一个健康副本还是所有副本
- load_balancing: 配置同一分片的多个副本间的请求分发策略
- 分片键: 决定数据写入哪个分片的关键字段
- _shard_num: Distributed 表的虚拟列，标识数据所在分片编号

## 关联笔记
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 同为主题为 ClickHouse distributed join 的学习笔记，内容高度相关
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 同为 ClickHouse 引擎学习（MaterializedMySQL）
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 同为 ClickHouse Join 相关研究
