---
note: 01KJBZ3EQK7E6D34480VFBNXRD.md
title: 20231025 - 通过arthas获取OMS队列监控
indexed_at: 2026-03-05T09:05:56.072382+00:00
---

## 摘要
记录通过 Arthas vmtool 命令监控 OMS 数据同步组件各类队列的方法。包含 Oracle 到 OceanBase 的同步环境连接信息及多个关键队列的监控命令示例。

## 关键概念
- Arthas: Alibaba 开源的 Java 应用诊断工具，支持运行时监控类实例状态
- vmtool: Arthas 命令，用于获取 JVM 中指定类的实例并执行表达式
- OracleLogEntryQueue: OMS 中存储 Oracle 日志条目的内存队列
- BridgeTaskManager: OceanBase 连接器中管理桥接任务的线程管理器
- 队列监控: 通过 vmtool 的--express 参数获取队列大小、积压量等指标

## 关联笔记
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: 同样使用 Arthas 分析 OMS 全量复制链路的队列状态
- 01KJBZ4E6HJKFS7FGETVFA76RD.md: OMS 全量验证链路分析，涉及类似的 Arthas 诊断场景
- 01KJBZ45XPJ5T7FYJVGSDF5EKH.md: OMS 全量复制链路过程记录，同一项目的相关笔记
