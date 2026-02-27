---
note: 01KJBZ2VNFT3ADNQ91QVJNYEG4.md
title: 20230828 - Oceanbase redo log (clog) 阅读
indexed_at: 2026-02-27T10:34:28.066922+00:00
---

用户希望我分析这篇关于 OceanBase redo log (clog) 的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 OceanBase 数据库 redo log (clog) 阅读分析的笔记
2. 主要内容包括：
   - 日志流阅读
   - __all_core_table 表的结构和定义
   - redo log 的具体内容分析
   - snapshot_gc_scn 的 UPDATE 事件分析

3. 核心概念包括：
   - redo log/clog
   - __all_core_table
   - tablet_id
   - K-V 结构
   - snapshot_gc_scn

4. 与其他笔记的关联：
   - 这篇笔记是关于 OceanBase 的 redo log
   - 其他笔记主要是关于 MySQL、Clickhouse、InnoDB 等
   - 可能关联的笔记：
     - 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理（都是关于数据库日志/刷盘机制）
     - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究（数据库刷页机制）
     - 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿（数据库刷盘）
   
   但这些主要是 MySQL 相关，与 OceanBase 的直接关联不大。

5. 标签应该包括：OceanBase, redo log, clog, 日志分析，__all_core_table 等

6. 摘要：这篇笔记分析了 OceanBase 的 redo log (clog) 结构，包括 __all_core_table 表的 K-V 存储设计和具体日志内容解析。

通过追踪 snapshot_gc_scn 的 UPDATE 操作，展现了日志中数据变更的具体过程。## 标签
OceanBase, redo log, clog, 日志分析，__all_core_table，K-V 存储

## 摘要
这篇笔记分析了 OceanBase 的 redo log (clog) 结构，包括 __all_core_table 系统表的 K-V 存储设计和表定义。通过具体日志样例解析了 INSERT 和 UPDATE 操作的 redo log 格式，梳理了 snapshot_gc_scn 的更新事件。

## 关键概念
- __all_core_table: OceanBase 系统表，采用 K-V 结构将二维关系表拆分成一维存储
- redo log (clog): OceanBase 的重做日志，记录数据变更操作
- tablet_id: OceanBase 中数据分片的标识符
- K-V 结构: __all_core_table 的存储格式，rowkey 包含 table_name、row_id、column_name
- snapshot_gc_scn: 全局统计信息中的快照垃圾回收位点

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同为数据库日志与刷盘机制研究（InnoDB 刷脏页）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同为数据库刷脏页机制研究
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同为数据库刷盘与日志相关研究
