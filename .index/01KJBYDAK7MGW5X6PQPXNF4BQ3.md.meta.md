---
note: 01KJBYDAK7MGW5X6PQPXNF4BQ3.md
title: 20210429 - SQLLancer 学习
indexed_at: 2026-03-05T07:24:39.105768+00:00
---

## 摘要
SQLLancer 是用于检测 SQL 处理逻辑错误的测试工具，采用三种逻辑推断方法进行验证。核心方法包括 Ternary Logic Partitioning（查询拆分对比）、Non-optimizing Reference Engine Construction（优化器正确性测试）和 Pivoted Query Synthesis（随机行验证）。

## 关键概念
- Ternary Logic Partitioning: 将 Query 拆分成多个子 Query，合并结果后与原始结果对比验证
- Non-optimizing Reference Engine Construction: 测试优化前和优化后 SQL 的区别，验证优化器正确性
- Pivoted Query Synthesis: 随机选取一行构造条件，检查 SQL 结果是否包含该行
- Query Partition: 查询分区，Ternary Logic Partitioning 的等价概念

## 关联笔记
无
