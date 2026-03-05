---
note: 01KJBYQSJVES1CY4G5GA5WTKYF.md
title: 20221021 - clickhouse mutations运行代码分析
indexed_at: 2026-03-05T08:10:32.571367+00:00
---

## 摘要
分析 ClickHouse 20.1.8.41 版本中 ReplicatedMergeTree 表引擎的 mutations 执行流程，涵盖从 ZooKeeper 获取 mutations 到执行 log entry 的完整链路。涉及 updateMutations、mergeSelectingTask、queueTask 三个核心任务的协作机制。

## 关键概念
- ReplicatedMergeTreeQueue::updateMutations: 从 ZooKeeper 获取 mutations 并分发到内存结构 mutations_by_znode 和 mutations_by_partition
- StorageReplicatedMergeTree::mergeSelectingTask: 根据 mutation 生成 log entry 的调度任务
- StorageReplicatedMergeTree::queueTask: 从 queue 获取 log entry 并执行，处理 MUTATE_PART 类型
- mutations_by_partition: 按 partition 组织的 mutations 内存结构，用于快速获取 commands
- tryExecutePartMutation: 执行 part 变异的核心函数，从 mutations_by_partition 获取 commands

## 关联笔记
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: 同一时期 ClickHouse mutations 问题排查，涉及 AST is too big 报错和 system.mutations 分析
- 01KJBYVTTVNYTSRBX5JBT2MJKP.md: 涉及 MUTATE_PART log entry 执行和 ReplicatedMergeTreeQueue::shouldExecuteLogEntry 逻辑
- 01KJBYWMG762P36NB4CPT3P64J.md: 同样涉及 mutations 执行时 part size 阈值判断的日志分析
