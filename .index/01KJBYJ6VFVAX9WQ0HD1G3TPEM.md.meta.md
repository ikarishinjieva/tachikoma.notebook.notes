---
note: 01KJBYJ6VFVAX9WQ0HD1G3TPEM.md
title: 20220419 - 国债故障
indexed_at: 2026-03-05T07:57:59.798993+00:00
---

## 标签
MySQL, 故障分析, SYS_TABLES, buffer pool, 刷盘队列, 锁定页

## 摘要
MySQL 5.7.29 中 SYS_TABLES 系统表的 125 万数据页被异常设置为锁定页，无法从 buffer pool 刷盘，导致刷盘队列阻塞。dict_stats_save 线程在持有数据字典锁时等待刷盘完成，引发复制延迟和性能问题。

## 关键概念
- buffer pool 锁定页: 不能被刷出内存的数据页，正常情况下应被释放
- flush_list: InnoDB 刷盘队列，按 LSN 顺序组织待刷盘的数据页
- dict_stats_save: InnoDB 后台线程，负责将统计信息持久化到数据字典表
- innodb_temporary: SYS_TABLES 系统表默认刷盘的目标表空间
- buf_flush_wait_flushed: 等待指定 LSN 之前的数据页完成刷盘的函数

## 关联笔记
- 01KJBYJCDGQG0YRF8688HMW9VH.md: 同一故障的后续深入分析，追踪 SYS_TABLES 锁定页泄漏的根本原因
- 01KJBYFQ942KXQ4NR803SNC0T2.md: 国债项目 MySQL 性能相关笔记，涉及相同业务场景
