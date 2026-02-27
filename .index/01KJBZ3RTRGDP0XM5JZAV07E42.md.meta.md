---
note: 01KJBZ3RTRGDP0XM5JZAV07E42.md
title: 20231108 - 整理OMS的增量复制链路
indexed_at: 2026-02-27T10:38:09.496548+00:00
---

用户要求我分析这篇关于 OMS 增量复制链路的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 Oracle OMS（OceanBase Migration Service）增量复制链路的技术笔记
2. 主要内容是描述从 Oracle 数据库通过 Logminer 订阅 redo log，然后进行分析、序列化的整个流程
3. 涉及的核心概念包括：Logminer、redo log、OracleLogEntry、OracleLogRecord、事务处理等
4. 笔记日期是 20231108

关于关联笔记，我需要查看其他笔记列表中是否有相关的：
- 大部分笔记是 2021 年的，主题涉及 MySQL、Clickhouse、性能分析等
- 这篇笔记是关于 Oracle Logminer 和 OMS 的
- 看起来没有直接关联的笔记（没有 Oracle/Logminer/OMS 相关的）

让我生成索引：

标签：OMS, Oracle, Logminer, 增量复制, redo log, 数据同步

摘要：这篇笔记详细记录了 OMS 增量复制链路从 Oracle 数据库读取 redo log 的完整流程。包括 Logminer 订阅、redo log 分析、事务信息识别三个核心阶段，以及各阶段的线程模型和队列设计。

关键概念：
- Logminer: Oracle 提供的日志挖掘工具，用于读取和解析 redo log
- OracleLogEntry: Logminer 读取的原始日志项结构


- OracleLogRecord: 经过分析后的日志记录结构
- LogEntryFetcher: 负责从 LogMiner 读取 redo log 的线程
- OracleLogExtractor: 负责将 redo log 项分析转换为 OracleLogRecord 的线程

关联笔记：无（其他笔记主要是 MySQL、Clickhouse 相关，没有 Oracle/Logminer 主题）
## 标签
OMS, Oracle, Logminer, 增量复制, redo log, 数据同步

## 摘要
这篇笔记详细记录了 OMS 增量复制链路从 Oracle 数据库读取 redo log 的完整流程。包括 Logminer 订阅、redo log 分析、事务信息识别三个核心阶段，以及各阶段的线程模型、队列设计和关键代码调用栈。

## 关键概念
- Logminer: Oracle 提供的日志挖掘工具，用于读取和解析 redo log
- OracleLogEntry: Logminer 读取的原始日志项结构
- OracleLogRecord: 经过分析转换后的日志记录结构
- LogEntryFetcher: 负责从 LogMiner 读取 redo log 的线程
- OracleLogExtractor: 负责将 redo log 项分析转换为 OracleLogRecord 的线程

## 关联笔记
无
