---
note: 01KJBYVTTVNYTSRBX5JBT2MJKP.md
title: 20221123 - 光大 AST is too big 报错后续
indexed_at: 2026-03-05T08:17:06.314023+00:00
---

## 标签
ClickHouse, mutation, 后台线程池，参数调优，ReplicatedMergeTree, 故障诊断

## 摘要
记录了光大 ClickHouse 集群中 alter table modify setting 参数变更未生效的问题诊断过程。通过分析日志发现后台线程池繁忙导致 mutation 无法执行，max_source_parts_size 为 0 是根本原因，提供了扩大线程池或降低空闲线程阈值的解决方案。

## 关键概念
- ReplicatedMergeTreeQueue: 负责管理复制表的日志队列和 mutation 执行调度
- background_pool_size: 后台线程池大小，决定并发合并/变更能力
- number_of_free_entries_in_pool_to_execute_mutation: 执行 mutation 所需的空闲线程数阈值
- shouldExecuteLogEntry: 判断日志 entry 是否可执行的核心函数
- MUTATE_PART: mutation 操作的日志 entry 类型

## 关联笔记
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: 前继笔记，记录同一集群 Too many parts 报错及 mutations 积压问题
