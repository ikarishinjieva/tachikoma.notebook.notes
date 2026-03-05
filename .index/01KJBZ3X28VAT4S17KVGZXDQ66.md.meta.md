---
note: 01KJBZ3X28VAT4S17KVGZXDQ66.md
title: 20231115 - Oracle logminer学习
indexed_at: 2026-03-05T09:17:12.716358+00:00
---

## 标签
Oracle, LogMiner, OMS, Arthas, 增量同步, V$LOGMNR_CONTENTS

## 摘要
记录使用 Arthas 工具分析 OMS 系统调用 Oracle LogMiner 的行为，发现空闲压力下每 20 秒轮询 V$LOGMNR_CONTENTS 的机制。包含 LogEntryPrimaryFetcher 线程的 SQL 执行堆栈和调用链路分析。

## 关键概念
- V$LOGMNR_CONTENTS: LogMiner 解析 redo log 后返回的事务内容视图
- COMMITTED_DATA_ONLY: LogMiner 参数，仅返回已提交事务并按提交顺序排列
- LogEntryPrimaryFetcher: OMS 中负责从 redo log 获取日志条目的线程
- DBMS_LOGMNR: Oracle 提供的 LogMiner PL/SQL 包

## 关联笔记
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 通过 Arthas vmtool 命令监控 OMS LogMiner 队列状态
- 01KJBZ3P2X276MPVMQ02101059.md: OMS 增量复制链路的 LogMiner 日志分析与异常堆栈
- 01KJBZ446ZAMCGR9NRFN1S50TF.md: Oracle 并发压力下 OMS LogMiner 性能调优配置
