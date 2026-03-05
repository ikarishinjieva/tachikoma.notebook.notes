---
note: 01KJBYSTE6KE2T030VC4M9TP90.md
title: 20221109 - OMA评估
indexed_at: 2026-03-05T08:14:28.179984+00:00
---

## 标签
OMA, OceanBase, 数据库迁移, MySQL, 兼容性评估, general log

## 摘要
记录使用 OMA 工具评估 Confluence 数据库从 MySQL 5.7 迁移到 OceanBase 2.2.x 的兼容性分析配置。包含评估命令参数和 MySQL general log 连接追踪日志用于排查问题。

## 关键概念
- OMA: OceanBase Migration Assistant，数据库迁移兼容性评估工具
- SOURCE_TARGET 模式: 同时分析源库和目标库的兼容性规则
- general log: MySQL 全量 SQL 日志，用于追踪数据库连接和查询行为
- OBMYSQL: OceanBase 的 MySQL 兼容模式
- MySQL Connector Java: Java 连接 MySQL 的驱动程序

## 关联笔记
- 01KJBYT2C9Y03Q2BGZAVXP0HE8.md: OMA 参数列表整理，详细解释 ANALYZE 模式和 evaluate-mode 参数含义
- 01KJBYWKGYZ1ZAP7STRSZWAVZJ.md: OMS checker 代码分析，涉及 OceanBase 迁移后的数据校验工具
- 01KJBYFF2KJAHSJFDW23BMQCHV.md: MySQL crash 排查笔记，提到开启 general log 定位问题 SQL 的方法
