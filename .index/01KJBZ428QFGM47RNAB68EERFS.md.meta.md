---
note: 01KJBZ428QFGM47RNAB68EERFS.md
title: 20231123 - 在Oracle并发压力中, 测试OMS性能 [3]
indexed_at: 2026-03-05T09:19:28.702269+00:00
---

## 标签
OMS, Oracle, 性能调优, ConflictBrokerV2, TPCC, 数据同步

## 摘要
记录在 Oracle 并发压力测试场景下对 OMS 数据同步的性能调优配置，包括源端 logfetcher 间隔、目标端队列容量、JVM 内存等参数调整。详细列出手工创建 TPCC 表结构的 SQL，避免 IoT 表问题，并启用 ConflictBrokerV2 热点表合并机制。

## 关键概念
- ConflictBrokerV2: OMS 中带有热点表事务合并功能的冲突检测器
- hotKeyMerge: 启用热点表事务合并的配置参数
- TPCC: 数据库性能基准测试模型，包含 WAREHOUSE、DISTRICT、CUSTOMER 等表

## 关联笔记
- 01KJBZ449CMKM8CQ045S94KF5E.md: 同一系列性能测试的第 1 篇（20231117），场景 1 的测试记录
- 01KJBZ446ZAMCGR9NRFN1S50TF.md: 同一系列性能测试的第 4 篇（20231127），10 warehouse 场景的调优
- 01KJBZ406P2QGR92F58403PYZ1.md: ConflictBrokerV2 原理详解，包含参数说明和逻辑分析
