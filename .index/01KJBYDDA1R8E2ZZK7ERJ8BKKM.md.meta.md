---
note: 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md
title: 20210622 - Clickhouse distributed join 学习
indexed_at: 2026-03-05T07:30:29.794735+00:00
---

## 摘要
对比分析 ClickHouse 中 `distributed join distribute` 与 `distribute global join distribute` 两种执行模式的差异。前者右表查询执行 N 次，后者右表仅查询一次并广播到各节点内存表。提供 GLOBAL JOIN 运维建议：右表加 DISTINCT 减少数据量、部署同数据中心降低网络成本、调整数据分布避开 GLOBAL IN。

## 关键概念
- distributed_product_mode: 控制分布式表 JOIN 时如何处理跨分片数据
- GLOBAL JOIN: 右表只查询一次，结果广播到所有节点进行本地 JOIN
- Memory 临时表: GLOBAL JOIN 时右表结果在各节点内存中的存储形式
- 分布式聚合: 各节点本地 JOIN 完成后进行最终结果汇总
- max_parallel_replicas: 与分布式查询并行度相关的配置参数

## 关联笔记
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 同主题 ClickHouse Join 研究笔记，包含 GLOBAL JOIN 流程待办事项
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: ClickHouse 生产环境故障分析，涉及分布式表与复制表问题
