---
note: 01KJBZ3RTRGDP0XM5JZAV07E42.md
title: 20231108 - 整理OMS的增量复制链路
indexed_at: 2026-03-05T09:15:09.485550+00:00
---

## 摘要
详细梳理 OMS 从 Oracle 源库通过 Logminer 订阅 redo log 实现增量数据复制的完整链路，包含 8 个核心处理阶段及各阶段的线程、队列、数据结构。分析了从日志抓取、事务分析、序列化到输出的全流程，并指出 4 线程分析顺序取出导致的效率问题。

## 关键概念
- LogEntryFetcher: 从 Oracle LogMiner 读取 redo log 日志项的抓取线程
- OracleLogExtractor: 将 redo log 项分配给 4 个分析线程进行解析的组件
- LogRecordSerializer: 识别事务边界并填充事务信息的序列化线程
- OracleLogEntry/OracleLogRecord: Logminer 处理过程中的核心数据结构
- 队列管道: 各处理阶段通过命名队列 (如 LogEntryFetcher_OutQueue) 传递数据

## 关联笔记
- 01KJBZ3X28VAT4S17KVGZXDQ66.md: 同一作者的 Oracle logminer 学习，详细记录 logminer 官方文档和 SQL 行为分析
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: 姊妹篇笔记，整理 OMS 全量复制链路 (基于 ROWID 分块扫描而非 logminer)
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 通过 arthas 监控 OMS 队列，包含本笔记所述各队列的监控命令
