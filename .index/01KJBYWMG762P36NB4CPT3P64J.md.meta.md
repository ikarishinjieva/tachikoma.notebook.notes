---
note: 01KJBYWMG762P36NB4CPT3P64J.md
title: 20221208 - 光大CK DELETE数据失败
indexed_at: 2026-03-05T08:18:22.690151+00:00
---

## 标签
ClickHouse, DELETE, DDL, mutation, ReplicatedMergeTree, 故障排查

## 摘要
ClickHouse 集群执行 DELETE DDL 后数据未删除，原因是 mutations_sync 使用默认值 0，DDL 不等待 mutation 回放即返回成功。mutation 回放失败因 source parts size 超过当前最大值限制，导致 MUTATE_PART 无法执行。

## 关键概念
- mutations_sync: 控制 DDL 是否等待 mutation 完成的参数，默认值 0 表示不等待
- DDLWorker: ClickHouse 处理 DDL 语句的工作线程，负责在集群上执行 DDL
- ReplicatedMergeTreeQueue: 复制表的队列管理模块，负责处理 mutation 等日志条目
- MUTATE_PART: mutation 回放时对数据分区执行的实际变更操作

## 关联笔记
- 01KJBYVTTVNYTSRBX5JBT2MJKP.md: 相同的"source parts size"报错，详细分析 mutation 回放失败原因
- 01KJBYXQMP1JBTG1M6CHNYWF8E.md: 同一 CK 集群的 mutation 积压问题，涉及 CK_BLACK_FILE_LOCAL 表
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: AST is too big 报错，同一集群的 mutation 相关问题
