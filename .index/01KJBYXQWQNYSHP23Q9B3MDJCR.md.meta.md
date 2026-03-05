---
note: 01KJBYXQWQNYSHP23Q9B3MDJCR.md
title: 20221020 - 光大clickhouse Too many parts报错
indexed_at: 2026-03-05T08:29:09.427180+00:00
---

## 标签
ClickHouse, AST is too big, 复制表，故障诊断，StorageReplicatedMergeTree, 光大

## 摘要
分析光大 ClickHouse 集群两台服务器 (26 和 27) 的日志，定位 27 节点在复制表中报出"AST is too big"错误。该错误导致复制队列阻塞，造成数据延迟。

## 关键概念
- StorageReplicatedMergeTree: ClickHouse 复制表存储引擎，负责数据复制和合并操作
- AST is too big: 抽象语法树节点数超过限制 (500000) 的错误，会阻塞 mutation 执行
- ReplicatedMergeTreeQueue: 复制表的日志队列，负责回放 ZooKeeper 中的复制操作日志
- queueTask(): 后台处理队列任务的函数，负责执行日志中的合并/变更操作

## 关联笔记
- 01KJBYXQPQ1SX1N9ZQFRYPFKGG.md: 同一工单 (BEIJ-2780) 的早期诊断，记录 Too many parts 报错的初始分析
- 01KJBYVTTVNYTSRBX5JBT2MJKP.md: 明确标注前继为本笔记，是 AST is too big 报错的后续跟进
- 01KJBYXQMP1JBTG1M6CHNYWF8E.md: 同一问题链的后续，记录调大后台线程数导致的内存问题
