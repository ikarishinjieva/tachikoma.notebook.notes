---
note: 01KJBYD9TKEMSF0Z4RF5DHT1QK.md
title: 20210404 - MySQL刷脏页研究
indexed_at: 2026-03-05T07:21:01.581782+00:00
---

## 标签
MySQL, InnoDB, 刷脏页, Adaptive Flushing, redo log, buffer pool

## 摘要
详细分析 InnoDB 三种刷脏机制：脏页比例触发、Free List 不足触发、redo log 空闲不足触发。重点讲解 Adaptive Flushing 算法原理，通过 redo log checkpoint age 动态计算刷盘 IO 量，使刷盘速度跟上事务生成速度。

## 关键概念
- Adaptive Flushing: 根据 redo log 未 checkpoint 的页长度动态评估刷脏策略，使 TAIL 刷盘速度追上 HEAD 事务生成速度
- innodb_max_dirty_pages_pct_lwm: 脏页预刷低水位，超过此值开始按比例刷脏，避免达到高水位后全力刷脏
- page cleaner: InnoDB 刷盘线程架构，coordinator 负责计算刷盘策略，worker 负责执行刷盘任务
- checkpoint age: redo log 中未刷脏页的长度，用于计算当前应刷盘的 IO 压力
- innodb_io_capacity_max: 刷盘 IO 上限，但开启 innodb_flush_sync 时 redo log checkpoint 刷脏可超过此限制

## 关联笔记
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同系列刷盘研究笔记，分析数据文件刷盘慢导致 SQL 卡顿的故障
- 01KJBYJ6VFVAX9WQ0HD1G3TPEM.md: 国债故障分析，涉及 buffer pool 刷盘队列 (flush_list) 被锁定页阻塞导致刷盘缓慢
- 01KJBYFQ942KXQ4NR803SNC0T2.md: MySQL 8.0 与 5.7 性能对比，涉及 redo log 写入机制差异
