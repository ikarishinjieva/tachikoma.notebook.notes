---
note: 01KJBYE4H01Z06Q70JXCFR0THQ.md
title: 20210811 - 天津滨银汽车Clickhouse问题咨询
indexed_at: 2026-03-05T07:39:37.042058+00:00
---

## 摘要
记录天津滨银汽车 ClickHouse 查询问题的复现与排查过程。在 ClickHouse 21.3.2.5 环境中搭建 MaterializeMySQL 引擎复制 MySQL 数据，复现了用户现场的查询报错问题，日志显示 StorageMaterializeMySQL 读取时出现异常。

## 关键概念
- MaterializeMySQL: ClickHouse 的 MySQL 数据库复制引擎，通过 binlog 同步数据
- ClickHouse 查询优化: MergeTreeWhereOptimizer 将条件移动到 PREWHERE 提升性能
- 堆栈分析: 通过日志定位 StorageMaterializeMySQL::read 方法异常

## 关联笔记
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 同样涉及 MaterializeMySQL 引擎的复制与崩溃问题分析
- 01KJBYFF2KJAHSJFDW23BMQCHV.md: 同一客户（滨银汽车）的 MySQL crash 问题记录
