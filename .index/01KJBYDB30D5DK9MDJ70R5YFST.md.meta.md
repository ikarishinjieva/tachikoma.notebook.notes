---
note: 01KJBYDB30D5DK9MDJ70R5YFST.md
title: 20210513 - 百胜cpu高的探索
indexed_at: 2026-03-05T07:26:43.675575+00:00
---

## 摘要
分析百胜环境 MySQL CPU 高的问题，通过 perf 截图和堆栈追踪定位到 PolicyMutex 锁竞争。发现 update 操作时四类堆栈均指向 trx0trx.cc 中 undo segment 申请时的锁等待。

## 关键概念
- PolicyMutex: InnoDB 的互斥锁机制，用于保护共享资源访问
- undo segment: 存储事务回滚信息的日志段，update 操作需申请可用段
- trx_assign_rseg: 为事务分配 redo log rollback segment 的函数
- lock_table: InnoDB 表锁获取流程，触发事务读写模式设置

## 关联笔记
- 01KJBZEE9JTH4RKQG664GFW88H.md: 同样涉及 PolicyMutex 和 trx0trx.cc 堆栈分析的 MySQL 问题排查
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: InnoDB 刷脏页机制研究，同属 InnoDB 内部机制分析
