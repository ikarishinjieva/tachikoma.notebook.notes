---
note: 01KJBYFD3950T3XNVP5R4BXYZV.md
title: 20211124 - 农行load data引发锁的分析
indexed_at: 2026-02-27T10:14:25.781574+00:00
---

用户需要我分析这篇关于 MySQL 死锁分析的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20211124 - 农行 load data 引发锁的分析
2. 内容涉及：MySQL InnoDB 锁机制、死锁分析、GAP 锁、READ UNCOMMITTED 隔离级别、gdb 调试

核心内容：
- 在 READ UNCOMMITTED 隔离级别下进行 insert 操作引发死锁
- 通过 performance_schema.data_locks 观察锁状态
- 发现 S,GAP 锁的存在
- 通过 gdb 调试发现 GAP 锁是由 lock_update_delete 等函数生成的
- 解释了删除行时会进行 gap 锁继承

关联笔记分析：
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 → 相关，因为笔记中使用了 performance_schema.data_locks
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 → 可能相关，涉及 InnoDB
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页 → 涉及 InnoDB

最相关的是 performance_schema 那篇，因为笔记中明确使用了 performance_schema 来观察锁。

现在生成索引元数据。
## 标签
MySQL, InnoDB, 死锁分析, GAP 锁, 隔离级别, gdb 调试

## 摘要
分析 READ UNCOMMITTED 隔离级别下 insert 操作引发死锁的现象，通过 performance_schema.data_locks 观察锁状态。通过 gdb 调试追踪 GAP 锁的生成堆栈，发现删除行时会进行 gap 锁继承。

## 关键概念
- READ UNCOMMITTED: 最低隔离级别，通常不应出现 GAP 锁但实际出现了
- GAP 锁: 间隙锁，用于保护记录之间的间隙防止幻读
- 锁继承: 删除行时 GAP 锁会传递给相邻记录
- performance_schema.data_locks: MySQL 8.0 用于观察当前锁状态的表

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 笔记中使用了 performance_schema.data_locks 观察锁状态
