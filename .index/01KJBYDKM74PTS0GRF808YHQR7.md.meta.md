---
note: 01KJBYDKM74PTS0GRF808YHQR7.md
title: 20210628 - DDL 过程梳理
indexed_at: 2026-03-05T07:31:09.428511+00:00
---

## 摘要
梳理 MySQL 8.0 中 DDL 操作的完整代码流程，重点分析 inplace/instant 两种优化策略的执行路径。涵盖从入口函数 mysql_alter_table 到引擎层的 prepare/alter/commit 三阶段，以及 online DDL 的日志回放机制。

## 关键概念
- inplace alter: 无需重建表的 DDL 执行方式，通过原地修改元数据或索引实现
- instant alter: MySQL 8.0 引入的快速 DDL，仅修改元数据不触碰数据页
- online DDL: 允许 DDL 执行期间并发 DML 操作，通过日志回放保证数据一致性
- HA_ALTER_INPLACE: 存储引擎返回的 inplace 支持级别，决定锁升级策略
- row_merge_build_indexes: 读取聚簇索引并用归并排序构建新索引的核心函数

## 关联笔记
- 01KJBYKCTPB0T5X02RSRVRS3W4.md: 包含 inplace_alter_table 的 crash 堆栈，展示 row_merge_build_indexes 调用链
- 01KJBZ64PQ0DMVAKYHABCZAV4K.md: 讨论 online DDL 期间 DML 日志回放可能导致 duplicate key 错误的场景
