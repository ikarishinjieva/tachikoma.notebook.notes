---
note: 01KJBYJDY946YJAPPA1981SMS8.md
title: 20220508 - 杭州银行 MySQL slave 崩溃
indexed_at: 2026-03-05T07:58:48.762909+00:00
---

## 标签
MySQL, InnoDB, 锁等待, 崩溃分析, semaphore, LOCK_SYS_WAIT

## 摘要
杭州银行 MySQL slave 因 InnoDB 锁等待超时崩溃，大量线程阻塞在 LOCK_SYS_WAIT  mutex，等待超过 900 秒后触发服务器 intentional crash。分析指向 lock_wait_timeout_thread 可能卡顿，但缺乏有效监控指标。

## 关键概念
- LOCK_SYS_WAIT: InnoDB 锁等待系统 mutex，保护锁等待表操作
- lock_wait_timeout_thread: 负责检测锁等待超时的后台线程
- semaphore wait: 信号量等待，InnoDB 用于线程同步，超过 600 秒触发崩溃
- Global_exclusive_latch_guard: InnoDB 全局排他锁，可能与锁等待超时检测相关
- INNODB MONITOR OUTPUT: InnoDB 诊断输出，用于分析锁等待状态

## 关联笔记
- 01KJBZEE9JTH4RKQG664GFW88H.md: 海南银行 MySQL crash 分析，同样涉及 InnoDB 锁等待和 trx_sys mutex 等待问题
