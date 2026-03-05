---
note: 01KJBZ2TD63SRBNM9GPKBXR5G4.md
title: 20230810 - OB如何增加GTID
indexed_at: 2026-03-05T08:57:53.082536+00:00
---

## 摘要
分析 OceanBase 如何通过 TX_ID 生成 GTID，调查 ObLogBR 日志项中的唯一标识来源。提出通过 TX_ID 二分查找 LSN 的方案，以解决备份恢复后日志传输起始点位问题。

## 关键概念
- TX_ID: 事务 ID，通过 GTI RPC 从 ObTransIDService 批量申请
- LSN: Log Sequence Number，日志序列号，用于订阅日志
- SCN: System Change Number，系统变更号，与时间戳关联
- ObLogBR: OceanBase 从 redo log 订阅的日志记录结构
- GTI: Global Transaction ID 服务，负责生成事务 ID

## 关联笔记
- 01KJBZ2VNFT3ADNQ91QVJNYEG4.md: 同为 OceanBase clog/redo log 分析笔记，涉及日志流和 SCN 机制
