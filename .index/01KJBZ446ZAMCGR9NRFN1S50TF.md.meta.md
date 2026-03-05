---
note: 01KJBZ446ZAMCGR9NRFN1S50TF.md
title: 20231127 - 在Oracle并发压力中, 测试OMS性能 [4]
indexed_at: 2026-03-05T09:20:40.940317+00:00
---

## 标签
OMS, Oracle, 性能调优, TPCC, 数据同步, ConflictBrokerV2

## 摘要
记录在 Oracle 并发压力测试场景下对 OMS 数据同步性能的多项调优措施，包括源端 logminer 配置、目标端队列阈值、JVM 内存、sink 并发度等参数调整。通过 ConflictBrokerV2 解决热点表冲突问题，并手工创建 TPCC 表避免 IoT 表问题。

## 关键概念
- ConflictBrokerV2: OMS 的热点键合并机制，用于处理高并发场景下的行冲突
- logminer: Oracle 日志挖掘工具，OMS 用于捕获源端增量数据
- sink.workerNum: 目标端写入并发度配置，影响数据回放效率
- coordinator.maxRecordCapacity: 队列准入阈值，控制内存中可缓存的记录数

## 关联笔记
- 01KJBZ449CMKM8CQ045S94KF5E.md: 同一系列性能测试的第一篇，记录 ConflictBrokerV1 的问题诊断过程
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: 后续整理的 OMS 全量复制链路文档，涉及相同的 TPCC 表结构
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 使用 arthas 监控 OMS 队列的相关技术，与本笔记的调优方法相关
