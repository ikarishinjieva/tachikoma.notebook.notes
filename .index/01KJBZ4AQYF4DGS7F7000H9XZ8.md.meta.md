---
note: 01KJBZ4AQYF4DGS7F7000H9XZ8.md
title: 20231204 - 整理OMS的全量复制链路
indexed_at: 2026-03-05T09:23:08.269047+00:00
---

## 摘要
记录 OMS 全量复制链路的两个核心处理器：扫描表元数据获取表结构信息，以及数据切分器通过 Block/PK 等方式对数据进行边界切分。包含完整的调用栈、队列输出、Arthas 监控命令及 Oracle 块号查询 SQL 样例。

## 关键概念
- SliceTaskManager: 管理表扫描任务分配和切片队列输出的核心组件
- DataFlowSliceProvider: 提供数据切分边界并管理切片队列的提供者
- Block 切分: 基于 Oracle DBA_EXTENTS 的块号 (OMS_BLOCK_NUMBER) 进行数据范围切分
- Checkpoint: 记录切片进度的检查点机制，用于断点续传

## 关联笔记
- 01KJBZ4E6HJKFS7FGETVFA76RD.md: 全量验证链路笔记，与全量复制链路形成完整流程
- 01KJBZ45XPJ5T7FYJVGSDF5EKH.md: 全量复制链路整理过程笔记，记录 checkpoint 配置细节
- 01KJBZ3EQK7E6D34480VFBNXRD.md: Arthas 获取 OMS 队列监控方法，与本笔记的 Arthas 命令相呼应
