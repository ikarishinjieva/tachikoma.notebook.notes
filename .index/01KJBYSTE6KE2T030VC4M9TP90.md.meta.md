---
note: 01KJBYSTE6KE2T030VC4M9TP90.md
title: 20221109 - OMA评估
indexed_at: 2026-02-27T10:19:44.313982+00:00
---

用户要求我分析这篇关于 OMA 评估的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：20221109 - OMA 评估
2. 内容主要是关于评估 confluence 库的迁移工具配置和 MySQL general log 分析

关键信息：
- OMA 评估工具（数据库迁移评估工具）
- 从 MySQL 5.7 迁移到 OBMYSQL 2.2.x（OceanBase MySQL 模式）
- 使用了 ANALYZE 模式
- 开启了 general log 来观察连接和查询行为
- 涉及 confluence 数据库

标签应该是：OMA, 数据库迁移，MySQL, OceanBase, 评估

摘要：记录使用 OMA 工具评估 confluence 库从 MySQL 5.7 迁移到 OceanBase MySQL 模式的配置参数。通过开启 general log 分析客户端连接和查询行为。

关键概念：
- OMA: 数据库迁移评估工具
- ANALYZE 模式：评估分析模式
- general log: MySQL 通用查询日志
- OBMYSQL: OceanBase MySQL 兼容模式

关联笔记：看其他笔记列表，有几篇相关的：
- 20210414 - 华夏银行迁移工具评估（也是迁移工具评估）
- 20210525 - 临时磁盘表，INFORMATION_SCHEMA.FILES 和 performance_schema（涉及 performance_schema）


- 20210330 - performance_schema 的日常使用（涉及 performance_schema）

最相关的是华夏银行迁移工具评估那篇，因为都是关于数据库迁移工具评估的内容。
## 标签
OMA, 数据库迁移, MySQL, OceanBase, 评估, General Log

## 摘要
记录使用 OMA 工具评估 confluence 库从 MySQL 5.7 迁移到 OceanBase MySQL 2.2.x 的配置参数。通过开启 MySQL general log 分析客户端连接行为和查询语句。

## 关键概念
- OMA: 数据库迁移评估工具，支持 SOURCE_TARGET 模式分析源库与目标库兼容性
- ANALYZE 模式：OMA 的评估分析模式，用于收集源库元数据和 SQL 特征
- OBMYSQL: OceanBase 的 MySQL 兼容模式，目标数据库类型
- General Log: MySQL 通用查询日志，记录所有客户端连接和执行的 SQL 语句

## 关联笔记
- 01KJBYDACWHS9VPYASX7JF6Q61.md: 同为数据库迁移工具评估笔记（华夏银行项目）
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 涉及 INFORMATION_SCHEMA 和 performance_schema 观测相关
