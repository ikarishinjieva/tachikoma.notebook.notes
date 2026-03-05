---
note: 01KJBZ3VXDMGF2BT3VGE9QTJ4X.md
title: 20231112 - OMS, 使用arthas将其处理队列中的元素导出
indexed_at: 2026-03-05T09:16:31.374203+00:00
---

## 标签
arthas, OMS, 队列监控, Oracle, LogMiner, vmtool

## 摘要
记录使用 Arthas 的 vmtool 命令导出 Oracle LogMiner 处理队列中积压的 OracleLogEntry 数据。同时提供多个队列入口点的 watch 监听命令，用于追踪数据在各处理队列间的流转时序。

## 关键概念
- vmtool: Arthas 提供的 JVM 工具接口，可获取堆中对象实例并执行 OGNL 表达式
- OracleLogEntryQueue: OMS LogMiner 组件中存储 Oracle redo log 解析记录的内存队列
- watch 命令: Arthas 方法监控命令，可捕获方法调用时的参数、返回值和执行时间
- OracleBackRecordQueue: 存储需要回表查询的日志记录队列

## 关联笔记
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 同样是使用 arthas 获取 OMS 队列监控状态，包含更多队列的 vmtool 查询示例
- 01KJBZ3RTRGDP0XM5JZAV07E42.md: 详细整理 OMS 增量复制链路中各组件和队列的数据流转关系
