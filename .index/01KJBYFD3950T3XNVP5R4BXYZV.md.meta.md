---
note: 01KJBYFD3950T3XNVP5R4BXYZV.md
title: 20211124 - 农行load data引发锁的分析
indexed_at: 2026-03-05T07:52:29.378612+00:00
---

## 摘要
分析 MySQL 在 read uncommitted 隔离级别下 insert 操作引发死锁的现象，通过 data_locks 表观察到意外出现的 S,GAP 锁。通过 gdb 调试追踪堆栈，发现 GAP 锁源于删除行时的 lock_rec_inherit_to_gap 锁继承机制。

## 关键概念
- lock_rec_inherit_to_gap: 删除记录时将原记录持有的锁转移到右侧记录的 gap 锁继承机制
- REC_NOT_GAP: 记录锁类型，锁定具体记录而非间隙
- S,GAP: 共享间隙锁，在 RUC 隔离级别下意外出现
- purge 机制: InnoDB 后台清理已删除记录版本的过程，触发锁继承

## 关联笔记
- 01KJBZED41K5CMAYSC5QKC2E76.md: 同属 MySQL 锁机制分析，涉及 mutex 锁的 signal_count 机制
- 01KJBZEE9JTH4RKQG664GFW88H.md: 涉及 data_lock_waits 表和 InnoDB 锁等待分析
- 01KJBZARNH5DADCGZBKQAHK6PD.md: MySQL crash coredump 分析，涉及 InnoDB 引擎堆栈追踪
