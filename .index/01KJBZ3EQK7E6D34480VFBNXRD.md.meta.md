---
note: 01KJBZ3EQK7E6D34480VFBNXRD.md
title: 20231025 - 通过arthas获取OMS队列监控
indexed_at: 2026-02-27T10:36:47.238601+00:00
---

用户要求我分析这篇关于通过 arthas 获取 OMS 队列监控的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主要内容**：
   - OMS 服务器的连接信息（服务器、容器、数据库连接等）
   - 如何使用 arthas 工具获取 Java 进程的队列监控信息
   - 具体的监控命令示例（vmtool 命令）
   - 涉及两个进程：ConnectDRCDeliver 和 Bootstrap

2. **核心概念**：
   - Arthas - Java 诊断工具
   - OMS 队列监控
   - vmtool 命令
   - Oracle LogMiner
   - 各种队列（OracleLogEntryQueue, OracleLogRecordQueue 等）

3. **标签**：
   - Arthas
   - OMS
   - 队列监控
   - Java 诊断
   - Oracle
   - 数据同步

4. **关联笔记**：
   查看其他笔记列表，主要是关于 MySQL、Clickhouse、数据库相关的笔记。这篇笔记主要关于 OMS（OceanBase Migration Service）和 Arthas 监控，与列表中的笔记关联度较低。最可能有关联的可能是：
   - 01KJBYDACWHS9VPYASX7JF6Q61.md: 20210414 - 华夏银行迁移工具评估（都是关于数据迁移工具的）
   - 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（都涉及 Java/JDK 相关问题）
   
   但整体来看关联度都不高，可能应该写「无」或者选最相关的 1-2 个。

现在我来生成输出。## 标签
Arthas, OMS, 队列监控, Java 诊断, 数据同步, Oracle

## 摘要
记录通过 Arthas 工具监控 OMS 数据同步队列的方法，包含服务器连接信息和具体监控命令。重点使用 vmtool 命令获取各类队列（OracleLogEntryQueue、OracleLogRecordQueue 等）的大小和积压情况。

## 关键概念
- Arthas: Alibaba 开源的 Java 诊断工具，可动态监控 JVM 内部状态
- vmtool: Arthas 命令，用于获取 JVM 中指定类的实例并执行表达式
- OracleLogEntryQueue: OMS 中存储 Oracle 日志条目的队列
- BridgeTaskManager: OMS JDBC Connector 中负责任务调度的组件
- 队列积压监控: 通过 pending()/size() 等方法观测数据同步延迟

## 关联笔记
- 01KJBYDACWHS9VPYASX7JF6Q61.md: 同为主题银行数据迁移工具相关的技术调研
- 01KJBYD9T2SKQ815RG0XWSD236.md: 均涉及 Java/JDK 环境下数据库连接器的问题排查
