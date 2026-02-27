---
note: 01KJBZ3X28VAT4S17KVGZXDQ66.md
title: 20231115 - Oracle logminer学习
indexed_at: 2026-02-27T10:38:25.884738+00:00
---

用户要求我分析这篇关于 Oracle LogMiner 学习的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20231115 - Oracle logminer 学习
2. 主要内容：
   - Oracle 官方文档关于 DBMS_LOGMNR 的链接
   - 讨论了 COMMITTED_DATA_ONLY 参数的作用
   - 分析了 OMS（可能是 Oracle GoldenGate 或类似工具）使用 LogMiner 的行为
   - 使用 Arthas 工具监控 Oracle JDBC 驱动的 SQL 执行
   - 发现 OMS 每 20 秒轮询 V$LOGMNR_CONTENTS 视图

核心主题：Oracle LogMiner、OMS、数据同步/捕获

标签应该包括：Oracle, LogMiner, OMS, 数据同步, SQL 监控, Arthas

摘要：记录 Oracle LogMiner 官方文档参数说明，通过 Arthas 分析 OMS 使用 LogMiner 的行为模式。发现 OMS 在空闲状态下每 20 秒轮询 V$LOGMNR_CONTENTS 视图获取变更数据。

关键概念：
- LogMiner: Oracle 日志挖掘工具，用于解析 redo log
- V$LOGMNR_CONTENTS: LogMiner 内容视图，包含解析后的日志记录
- COMMITTED_DATA_ONLY: LogMiner 参数，仅返回已提交事务
- OMS: 使用 LogMiner 进行数据捕获的工具
- Arthas: Java 诊断工具，用于监控 SQL 执行


- 关联笔记方面，虽然笔记列表中有其他数据库相关内容，但这篇专注于 Oracle LogMiner 和 OMS 的特定主题，与其他笔记没有直接关联。## 标签
Oracle, LogMiner, OMS, 数据同步, Arthas, SQL 监控

## 摘要
记录 Oracle LogMiner 官方文档中 COMMITTED_DATA_ONLY 等参数的作用说明。通过 Arthas 工具分析 OMS 使用 LogMiner 的实际行为，发现其在空闲状态下每 20 秒轮询 V$LOGMNR_CONTENTS 视图获取变更数据。

## 关键概念
- LogMiner: Oracle 日志挖掘工具，用于解析 redo log 中的事务数据
- V$LOGMNR_CONTENTS: LogMiner 内容视图，包含解析后的 SQL_REDO、SCN、时间戳等日志记录
- COMMITTED_DATA_ONLY: LogMiner 参数，仅返回已提交事务并按提交顺序排列
- OMS: 使用 LogMiner 进行数据捕获和同步的工具
- Arthas: Java 诊断工具，用于监控和追踪 JDBC SQL 执行行为

## 关联笔记
无
