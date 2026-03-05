---
note: 01KJBZ3GDKGKZ4M8GWVG6SPK2Z.md
title: 20231027 - 南航 ActionDB 3.x和4.x执行计划不一样
indexed_at: 2026-03-05T09:08:54.280234+00:00
---

## 标签
南航, ActionDB, OceanBase, 执行计划, 表分区, 性能优化

## 摘要
记录南航 ActionDB 3.x 和 4.x 版本执行计划不一致问题。包含 act_transaction、act_event、act_membership 三张核心表的完整定义，表采用 OceanBase 数据库并按 Ins_Dt 日期进行月度范围分区。

## 关键概念
- OceanBase: 分布式数据库系统，支持 MySQL 兼容模式
- 范围分区: 按 Ins_Dt 日期列进行月度分区，从 1900 年到 2040 年
- LOCAL 索引: 分区表上的本地索引，每个分区独立维护
- Bloom Filter: 用于加速数据查询的概率型数据结构，此处禁用

## 关联笔记
- 01KJBZ4J03FH868Z4ZDKGPQRDY.md: 涉及 ActionDB 数据库服务器的审计日志开发项目
- 01KJBZP5QNWMWNBV0WVYNQH5QF.md: 提及 OceanBase/ActionDB 业务线的发展成果
