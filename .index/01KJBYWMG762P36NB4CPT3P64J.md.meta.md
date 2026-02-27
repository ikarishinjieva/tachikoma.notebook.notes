---
note: 01KJBYWMG762P36NB4CPT3P64J.md
title: 20221208 - 光大CK DELETE数据失败
indexed_at: 2026-02-27T10:21:12.593532+00:00
---

Let me analyze this note about ClickHouse DELETE operation failure and generate the index metadata.

Key points from the note:
1. ClickHouse version 20.1.8.41
2. DELETE operation failed on a 2-node ClickHouse cluster (26/27)
3. DDL was created correctly with mutation ID 0000064613
4. The issue is that mutations_sync is set to default 0, so DDL returns without waiting for mutation completion
5. Mutation playback failed because source parts size is greater than the current maximum (0.00 B)

Tags should include: ClickHouse, DELETE, DDL, mutation, 故障排查

Summary: This note documents a ClickHouse DELETE operation failure investigation on a 2-node cluster. The root cause is that mutations_sync=0 doesn't wait for mutation completion, and mutation playback is blocked due to size limits.

Key concepts:
- mutations_sync: Controls whether DDL waits for mutation completion
- DDLWorker: Handles distributed DDL queries in ClickHouse
- Mutation: ClickHouse's mechanism for data modification operations
- ReplicatedMergeTreeQueue: Manages replication queue for table parts

Related notes:
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: Clickhouse 集群 Distributed Engine 学习 (related to ClickHouse cluster)
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: Clickhouse MaterializedMySQL 学习 (ClickHouse related)
- 01KJBYDBWTJGYAJWBZ93943E5R.md: Clickhouse Join 研究 (ClickHouse related)
- 01KJBYDD58AJTW5NKT3GET7B89.md: 旧：Clickhouse+学习笔记 (ClickHouse learning notes)


- 01KJBYDKM74PTS0GRF808YHQR7.md: DDL 过程梳理 (DDL process)

The most relevant connections are the ClickHouse cluster notes and the DDL process documentation, since this issue involves distributed DDL execution and mutation handling across the cluster.
## 标签
ClickHouse, DELETE 失败, DDL, Mutation, 故障排查，集群复制

## 摘要
记录光大银行 ClickHouse 集群 DELETE 操作失败问题的排查过程。DDL 正确创建了 mutation，但因 mutations_sync 为默认值 0 不等待回放完成，且 replication 队列因 part size 限制无法执行 mutation。

## 关键概念
- mutations_sync: 控制 DDL 是否等待本节点 mutation 回放完成的配置参数
- DDLWorker: ClickHouse 中负责处理分布式 DDL 任务的组件
- Mutation: ClickHouse 对数据修改操作（DELETE/UPDATE）的内部实现机制
- ReplicatedMergeTreeQueue: 复制表的队列，负责管理 part 的合并与 mutation 回放

## 关联笔记
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: Clickhouse 集群 Distributed Engine 学习，同属 ClickHouse 集群相关
- 01KJBYDKM74PTS0GRF808YHQR7.md: DDL 过程梳理，同属 DDL 执行流程分析
- 01KJBYDD58AJTW5NKT3GET7B89.md: 旧：Clickhouse+学习笔记，同属 ClickHouse 技术笔记
