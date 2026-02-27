---
note: 01KJBYPH9KR9B4KT123DR6PJAF.md
title: 20221019 - clickhouse "table is in readonly mode" 诊断
indexed_at: 2026-02-27T10:18:53.459257+00:00
---

用户希望我分析一篇关于 ClickHouse "table is in readonly mode" 诊断的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：ClickHouse 表只读问题的诊断
2. **核心问题**：Table is in readonly mode 错误
3. **根本原因**：ZooKeeper Session timeout/expired 导致 ClickHouse 表进入只读状态
4. **时间线**：2022.10.12 10:47 - 15:42 期间出现只读状态
5. **技术分析**：
   - ZooKeeper 会话超时周期性出现（每 10 分钟）
   - BackgroundProcessingPool 的调度逻辑
   - ClickHouse 在 ZK 会话过期时将表置为只读
6. **可能原因**：ZK 集群响应慢/失去 leader，或网络延迟

**标签**：ClickHouse, ZooKeeper, 只读表, Session Expired, 故障诊断, 分布式数据库

**摘要**：分析 ClickHouse 表出现"readonly mode"错误的根本原因，定位到 ZooKeeper 会话超时导致。通过日志分析发现 ZK Session Expired 周期性出现，触发 ClickHouse 将表置为只读状态。

**关键概念**：
- StorageReplicatedMergeTree: ClickHouse 的复制表引擎，依赖 ZooKeeper 协调
- ZooKeeper Session Expired: ZK 会话超时，导致 ClickHouse 无法协调复制操作


- BackgroundProcessingPool: ClickHouse 后台处理线程池，负责执行复制队列任务
- 只读模式：ClickHouse 在 ZK 连接失败时自动进入的保护状态

**关联笔记**：
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 20210501 - Clickhouse 集群 Distributed Engine 学习（ClickHouse 集群相关）
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习（ClickHouse 引擎相关）
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究（ClickHouse 相关）

这几篇笔记都涉及 ClickHouse 的不同方面，可以作为参考。## 标签
ClickHouse, ZooKeeper, 只读表, Session Expired, 故障诊断, 分布式数据库

## 摘要
分析 ClickHouse 表出现"table is in readonly mode"错误的根本原因，定位到 ZooKeeper 会话超时导致。通过日志分析发现 ZK Session Expired 周期性出现（每 10 分钟），触发 ClickHouse 将表置为只读状态，直至网络或 ZK 环境恢复。

## 关键概念
- StorageReplicatedMergeTree: ClickHouse 的复制表引擎，依赖 ZooKeeper 协调复制操作
- ZooKeeper Session Expired: ZK 会话超时，导致 ClickHouse 无法协调并进入只读保护状态
- BackgroundProcessingPool: ClickHouse 后台处理线程池，错误时会逐步增加睡眠时间的调度逻辑

## 关联笔记
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 同为 ClickHouse 集群相关学习笔记
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 同为 ClickHouse 引擎机制研究笔记
