---
note: 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md
title: 20210501 - Clickhouse集群 Distributed Engine 学习
indexed_at: 2026-03-05T07:25:08.417310+00:00
---

## 摘要
介绍 Clickhouse Distributed Engine 的工作原理，包括查询分发、负载均衡、写入机制等核心特性。详细分析了 SELECT/INSERT 处理流程、系统配置参数及故障排查方法。

## 关键概念
- Distributed Engine: 逻辑表引擎，不实际存储数据，负责将请求分发到各物理分片
- internal_replication: 控制写请求发往 replica 的方式，true 时只发往第一个健康副本
- load_balancing: 同一 shard 多 replica 间的请求分发策略配置
- _shard_num: 虚拟列，标识数据所在的分片编号
- system.clusters: 查看集群配置和状态的系统表

## 关联笔记
- 01KJBZ1SS4R8E3CBNAFQKKFX32.md: Clickhouse 性能优化相关书籍笔记
- 01KJBYQSJVES1CY4G5GA5WTKYF.md: Clickhouse mutations 代码分析
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: Clickhouse Too many parts 报错排查
