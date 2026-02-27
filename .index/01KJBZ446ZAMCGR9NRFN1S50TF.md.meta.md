---
note: 01KJBZ446ZAMCGR9NRFN1S50TF.md
title: 20231127 - 在Oracle并发压力中, 测试OMS性能 [4]
indexed_at: 2026-02-27T10:39:42.487525+00:00
---

用户要求我分析这篇关于 Oracle 并发压力下 OMS 性能测试的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 OMS（OceanBase Migration Service）性能调优的笔记
2. 场景是 10 warehouse, 10 vu 的 TPCC 测试
3. 涉及多个调优参数：logfetcher 间隔、队列阈值、JVM 内存、sink worker 数量等
4. 包含 TPCC 表的 DDL 语句

我需要提取：
- 标签：3-6 个关键词
- 摘要：2-3 句话
- 关键概念：3-5 个
- 关联笔记：从提供的列表中找相关的

看其他笔记列表，主要是 MySQL、Clickhouse、性能调优相关的，但这篇笔记是关于 Oracle 和 OMS 的，关联度不高。可能没有直接关联的笔记。

让我生成输出：## 标签
OMS, 性能调优, Oracle, TPCC, 数据同步, 并发测试

## 摘要
记录在 Oracle 并发压力场景下对 OMS 数据同步进行的多项性能调优操作。包括调整源端 logfetcher 参数、目标端队列阈值、JVM 内存、sink worker 数量及 ConflictBrokerV2 配置等。

## 关键概念
- ConflictBrokerV2: 用于处理热点表合并的冲突解决机制
- logfetcher: 源端日志抓取组件，负责从 Oracle 抓取 redo log
- sink.workerNum: 目标端写入工作线程数，影响并发写入能力
- coordinator.maxRecordCapacity: 目标端队列最大容量，控制准入阈值

## 关联笔记
无
