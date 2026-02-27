---
note: 01KJBYJ6VFVAX9WQ0HD1G3TPEM.md
title: 20220419 - 国债故障
indexed_at: 2026-02-27T10:16:14.101350+00:00
---

用户希望我分析这篇关于"国债故障"的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是关于 MySQL 故障分析的笔记
2. 主要问题是 SYS_TABLES 系统表的数据页被设置为锁定页，导致刷盘缓慢
3. 涉及 InnoDB buffer pool、刷盘队列、dict_stats_save 线程等概念
4. 有详细的堆栈分析

现在我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从提供的列表中选择）

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理 → 与刷盘相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 → 与刷盘相关
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表... → 提到 innodb_table_stats，与笔记中的 dict_stats_save 相关

让我生成输出。## 标签
MySQL, InnoDB, 刷盘故障, buffer pool, SYS_TABLES, 性能分析

## 摘要
分析 MySQL 因 SYS_TABLES 系统表 125 万数据页被异常设置为锁定页，导致 buffer pool 刷盘队列阻塞的故障。锁定页堆积使刷盘线程需从队列尾端扫描至首个非锁定页，造成 dict_stats_save 线程等待刷盘完成而卡顿。

## 关键概念
- buffer pool 锁定页: 不能被刷出内存的数据页，异常堆积会阻塞刷盘队列
- 刷盘队列 (flush list): InnoDB 将脏页按 LSN 顺序组织，后台线程从此队列刷盘
- dict_stats_save: 保存表统计信息的后台线程，需等待刷盘完成后才能写入数据字典

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同样涉及 InnoDB 后台刷脏页机制的研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 直接关联 MySQL 刷脏页的研究主题
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 涉及 innodb_table_stats 表，与本笔记 dict_stats_save 操作同一张表
