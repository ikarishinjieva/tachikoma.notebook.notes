---
note: 01KJBYDB30D5DK9MDJ70R5YFST.md
title: 20210513 - 百胜cpu高的探索
indexed_at: 2026-02-27T10:03:01.173481+00:00
---

用户希望我分析这篇关于百胜 CPU 高的探索笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 MySQL 性能问题的排查笔记
2. 现象是 CPU 高，没有特定 SQL 出问题
3. 分析了 PolicyMutex 的入口，涉及 update 操作、undo segment 等
4. 有 perf 截图和堆栈分析

我需要根据内容生成：
- 标签：3-6 个关键词
- 摘要：2-3 句话
- 关键概念：3-5 个核心概念
- 关联笔记：从给定的笔记列表中找出相关的

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页 - 有关联，都是 InnoDB 内部机制
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 有关联
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 有关联
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: Update 是如何进行的 - 高度相关，因为这篇笔记涉及 update 操作的堆栈分析
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 可能有关联，因为涉及性能分析


- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: Update 是如何进行的 - 最相关，直接对应笔记中的 update 操作分析
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页机制 - 相关，涉及 InnoDB 内部工作机制
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 相关，同属 InnoDB 性能优化范畴## 标签
MySQL, CPU 性能, InnoDB, 堆栈分析, Update 操作, 锁竞争

## 摘要
记录百胜客户 MySQL CPU 高的问题排查过程，通过 perf 和 gdb 堆栈分析定位到 PolicyMutex 竞争热点。发现 update 操作时在申请 undo segment 阶段产生大量锁等待，涉及四类堆栈路径。

## 关键概念
- PolicyMutex: InnoDB 内部用于保护共享资源的互斥锁机制
- undo segment: 用于存储事务回滚信息的日志段，update 前需申请
- trx_assign_rseg: 事务分配 redo log rollback segment 的核心函数
- 多范围读取 (MRR): MySQL 优化范围查询的批量读取技术

## 关联笔记
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: 同为主题为 Update 操作内部机制的分析笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 涉及 InnoDB 刷脏页机制，与 InnoDB 内部调度相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同为 MySQL 刷脏页与 InnoDB 内部机制研究
