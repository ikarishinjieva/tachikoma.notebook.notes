---
note: 01KJBZ428QFGM47RNAB68EERFS.md
title: 20231123 - 在Oracle并发压力中, 测试OMS性能 [3]
indexed_at: 2026-02-27T10:39:11.279286+00:00
---

用户要求我分析一篇关于 Oracle 并发压力下 OMS 性能测试的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：Oracle 并发压力测试中 OMS（OceanBase Migration Service）性能调优
2. **核心内容**：
   - 场景：1 warehouse, 1 vu，buildschema 数据初始化，数据同步慢
   - 多个调优措施：logfetcher 间隔、队列阈值、JVM 内存、printSinkRecords 行为、ConflictBrokerV2
   - 手工创建 TPCC 表的 DDL

3. **标签**：应该包含 OMS、性能调优、Oracle、数据同步、TPCC 等关键词

4. **摘要**：概括调优场景和主要措施

5. **关键概念**：ConflictBrokerV2、logfetcher、队列阈值、JVM 调优、TPCC 等

6. **关联笔记**：查看提供的笔记列表，主要是 MySQL、Clickhouse、性能分析相关的笔记。这篇笔记是关于 Oracle/OMS 性能调优的，与列表中大部分笔记关联度不高。可能有些微关联的是性能分析相关的，但整体来看没有直接关联的笔记。

让我生成输出：
## 标签
OMS, 性能调优, Oracle, 数据同步, TPCC, JVM 调优

## 摘要
记录在 Oracle 并发压力测试场景下，针对 OMS 数据同步慢问题的多项调优措施。包括调整 logfetcher 间隔、队列阈值、JVM 内存、ConflictBrokerV2 配置，以及手工创建 TPCC 表避免 IoT 表问题。

## 关键概念
- ConflictBrokerV2: OMS 热点表合并处理机制，用于优化热点表同步性能
- logfetcher: 源端日志抓取组件，控制 SQL 执行间隔影响抓取频率
- coordinator.maxRecordCapacity: 目标端队列准入阈值，控制队列容量
- TPCC: 标准数据库性能测试基准，包含 WAREHOUSE、DISTRICT、ORDER 等核心表

## 关联笔记
无
