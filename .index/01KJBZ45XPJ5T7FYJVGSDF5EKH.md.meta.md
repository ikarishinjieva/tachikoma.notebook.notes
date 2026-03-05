---
note: 01KJBZ45XPJ5T7FYJVGSDF5EKH.md
title: 20231128 - 整理OMS的全量复制链路 的过程
indexed_at: 2026-03-05T09:22:55.055076+00:00
---

## 标签
OMS, 全量复制，connector, 数据同步，火焰图，Oracle

## 摘要
记录 OMS 全量复制链路的调试过程，包括 connector 进程配置、火焰图性能分析和主启动器流程。涉及 Oracle 数据库到目标库的数据同步，分析 checkpoint 检查点机制和切片任务管理。

## 关键概念
- Connector: OMS 数据同步的核心进程，负责源库到目标库的数据传输
- Checkpoint: 记录数据同步进度的检查点文件，用于断点续传
- SliceTask: 数据切片任务，将大表分块进行并行同步
- 火焰图: 性能分析工具，用于识别线程活跃度和性能瓶颈

## 关联笔记
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: 同一主题的后续详细整理，完整梳理全量复制链路各处理器流程
- 01KJBZ4E6HJKFS7FGETVFA76RD.md: 全量验证链路分析，与复制链路配合使用进行数据一致性校验
- 01KJBZ3EQK7E6D34480VFBNXRD.md: OMS 队列监控方法，使用 arthas 获取运行时队列状态
