---
note: 01KJBYDBWTJGYAJWBZ93943E5R.md
title: 20210531 - Clickhouse Join 研究
indexed_at: 2026-03-05T07:27:56.244228+00:00
---

## 标签
ClickHouse, JOIN, 分布式查询, GLOBAL JOIN, 查询优化, Hash Join

## 摘要
ClickHouse JOIN 操作核心要点：小表放右侧（hash join 机制）、先过滤聚合再 JOIN（谓词不下推）、分布式表 JOIN 需加 GLOBAL 避免查询放大。涵盖 ASOF JOIN、内存限制配置、join_algorithm 等关键知识点及执行计划分析实验。

## 关键概念
- GLOBAL JOIN: 右表只查询一次并分发到各节点，避免 N² 次查询放大
- Hash Join: CK 默认算法，左表为 probe table，右表为 build table 且被广播
- ASOF JOIN: 基于不完全相等条件（如时序数据）的 JOIN，使用 closest_match_cond 找最匹配记录
- join_overflow_mode: 内存超限时的行为控制（THROW 或 BREAK）
- 谓词不下推: CK 查询优化器较弱，JOIN 条件的谓词不会自动下推到子查询

## 关联笔记
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: Clickhouse distributed join 学习，详解 GLOBAL JOIN 与普通 JOIN 在分布式表上的执行流程差异
- 01KJBYDD58AJTW5NKT3GET7B89.md: Clickhouse 学习笔记，包含执行计划 explain 格式、稀疏索引等基础知识框架
