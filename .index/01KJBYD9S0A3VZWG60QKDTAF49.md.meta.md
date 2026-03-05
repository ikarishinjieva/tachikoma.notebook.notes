---
note: 01KJBYD9S0A3VZWG60QKDTAF49.md
title: 20210406 - innodb 如何后台不刷脏页, 前台会怎么处理
indexed_at: 2026-03-05T07:17:33.766290+00:00
---

## 标签
InnoDB, 刷脏页, checkpoint, page cleaner, MySQL, 调试

## 摘要
通过 gdb 调试和 InnoDB Monitor 监控，分析 InnoDB 后台刷脏页与前台 SQL 处理的交互机制。核心是 log_checkpoint_margin 检查点边界的 preflush 逻辑，以及 srv_flush_sync 强制唤醒 page cleaner 进行 IO burst 的触发条件。

## 关键概念
- log_checkpoint_margin: 检查是否需要同步预刷盘和检查点的边界阈值
- srv_flush_sync: SQL 处理中强制唤醒 page cleaner 进行突发 IO 的标志
- log_lsn_checkpoint_age: 当前 LSN 与检查点 LSN 的差值，用于判断刷盘紧迫性
- page cleaner: InnoDB 后台线程，负责将缓冲池中的脏页刷写到磁盘

## 关联笔记
- 01KJBYJ6VFVAX9WQ0HD1G3TPEM.md: 国债故障分析，展示了 log_checkpoint_margin 因 SYS_TABLES 锁定页导致刷盘阻塞的实际案例
