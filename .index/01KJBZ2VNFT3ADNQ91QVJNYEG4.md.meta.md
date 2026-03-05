---
note: 01KJBZ2VNFT3ADNQ91QVJNYEG4.md
title: 20230828 - Oceanbase redo log (clog) 阅读
indexed_at: 2026-03-05T08:58:19.141173+00:00
---

## 标签
OceanBase, clog, redo log, 日志流, __all_core_table, snapshot_gc_scn

## 摘要
分析 OceanBase 日志流 1006-1 的 clog 结构，定位 tablet_id=1 对应 __all_core_table 系统表。梳理 snapshot_gc_scn 的 UPDATE 事件，该字段由 ObFreezeInfoDetector 轮询更新，用于集群快照 SCN 管理。

## 关键概念
- __all_core_table: OceanBase 核心系统表，K-V 结构存储元数据，rowkey 包含 table_name、row_id、column_name
- tablet_id: 数据分片标识，tablet_id=1 对应系统核心表
- snapshot_gc_scn: 全局统计字段，记录快照 GC 的 SCN 值，由 ObFreezeInfoDetector 定期更新
- Mutator: redo log 中的行变更操作类型，包含 RowKey、NewRow/OldRow 等信息

## 关联笔记
- 01KJBZ2TD63SRBNM9GPKBXR5G4.md: 讨论 OB 日志流 (LogServiceID) 和 redo log 订阅机制，与日志流分析相关
- 01KJBZ4GNWYCPGNN5HXQ8PS58N.md: 分析 OceanBase 内部表结构定义，与 __all_core_table 表结构相关
