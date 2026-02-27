---
note: 01KJBYD9TKEMSF0Z4RF5DHT1QK.md
title: 20210404 - MySQL刷脏页研究
indexed_at: 2026-02-27T10:01:13.270002+00:00
---

用户希望我分析这篇关于 MySQL 刷脏页的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：MySQL InnoDB 刷脏页机制研究
2. 核心内容：
   - 三种刷脏原因：脏页比例、Free list 不足、redo log 空闲不足
   - Adaptive Flushing 机制
   - 相关参数配置
   - 错误日志分析

3. 关联笔记分析：
   - 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理 - 直接相关，都是关于 InnoDB 刷脏
   - 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 直接相关，都是关于 MySQL 刷盘
   - 其他笔记关联度较低

现在生成元数据。## 标签
MySQL, InnoDB, 刷脏页, Adaptive Flushing, 性能优化, redo log

## 摘要
本文研究 InnoDB 刷脏页的三种触发机制：脏页比例、Free list 不足、redo log 空闲不足。重点分析 Adaptive Flushing 算法如何协调 redo log 写入与刷盘速度，以及相关参数调优建议。

## 关键概念
- Adaptive Flushing: 根据 redo log 未 checkpoint 的页长度动态调整刷盘速度，使 TAIL 追上 HEAD
- innodb_max_dirty_pages_pct_lwm: 脏页比例低水位，超过此值开始预刷脏
- checkpoint age: redo log 已使用量，决定刷盘压力的关键指标
- page cleaner: InnoDB 后台线程，负责协调和执行刷脏任务

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同主题，研究后台不刷脏时前台的处理机制
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同主题，研究数据文件刷盘慢导致 SQL 卡顿的问题
