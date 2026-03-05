---
note: 01KJBYD9VE4NJQXE9P0VKM16Q4.md
title: 20210329 - MySQL刷盘研究 - 数据文件刷盘慢导致SQL卡顿
indexed_at: 2026-03-05T07:22:53.006090+00:00
---

## 摘要
分析 MySQL 数据文件刷盘延迟导致 SQL 卡顿的问题，通过堆栈追踪定位到 fil_flush 函数等待。研究了 innodb_data_pending_fsyncs 指标构成及 InnoDB 刷盘机制。

## 关键概念
- innodb_data_pending_fsyncs: InnoDB 待处理 fsync 总数，等于 log 刷盘 pending 数加表空间刷盘 pending 数
- fil_n_pending_log_flushes: 与 FIL_TYPE_LOG 相关的待刷盘数，等于 innodb_os_log_pending_fsyncs
- fil_n_pending_tablespace_flushes: 与 FIL_TYPE_TABLESPACE 相关的待刷盘数
- checkpoint: InnoDB 重做日志的检查点，标记已刷盘的数据位置
- fil_flush: InnoDB 刷盘核心函数，负责将数据页同步到磁盘

## 关联笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同系列笔记，研究 InnoDB 刷脏页机制和自适应刷盘算法
- 01KJBYJ6VFVAX9WQ0HD1G3TPEM.md: 相关故障分析，SYS_TABLES 锁定页导致刷盘队列扫描缓慢
- 01KJBYJEQN14KSK24XPHFC6S1M.md: page_cleaner 日志研究，与刷盘性能监控相关
