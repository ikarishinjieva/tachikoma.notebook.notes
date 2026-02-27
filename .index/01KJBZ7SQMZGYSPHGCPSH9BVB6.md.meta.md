---
note: 01KJBZ7SQMZGYSPHGCPSH9BVB6.md
title: 20240329 - ChatDBA在提示词中使用plan
indexed_at: 2026-02-27T10:49:29.345647+00:00
---

用户要求我分析一篇关于 ChatDBA 提示词中使用 plan 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

笔记主要内容包括：
1. 一个 DBA 专家 AI 的提示词模板
2. 多个 MySQL 故障处理案例的参考文档，包括：
   - 全球同方人寿-MySQL 锁等待问题处理
   - MySQL 死锁日志分析
   - MySQL 锁阻塞处理
   - 厦门航空-MGR 主从同步慢问题
   - 农行-应用系统反应慢问题（MyISAM 表锁问题）
   - 浦发-insert 语句慢分析

核心主题是：MySQL 数据库故障诊断、锁问题（死锁、锁等待、锁阻塞）、性能优化

现在我需要：
1. 提取 3-6 个标签
2. 写 2-3 句摘要
3. 列出 3-5 个关键概念
4. 从提供的笔记列表中找出 0-3 篇关联笔记

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 如何后台不刷脏页 - 与 InnoDB 相关，有一定关联
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 笔记中提到了 performance_schema 查询，有关联


- 01KJBYD9TKEMSF0Z4RF5FMDG44.md: MySQL 刷脏页研究 - 与 IO/刷盘相关，有一定关联
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 与 IO 瓶颈相关，有关联

最相关的应该是 performance_schema 那篇，因为笔记中明确使用了 performance_schema 来诊断锁问题。

另外刷盘研究的笔记也值得关联，因为 IO 瓶颈是笔记中提到的关键问题之一。
## 标签
MySQL, 锁等待, 死锁, 故障诊断, 性能优化, 提示词

## 摘要
本笔记整理了 ChatDBA 诊断 AI 的提示词模板，包含多个 MySQL 锁问题和性能故障的处理案例。涵盖 IO 瓶颈、死锁日志分析、锁阻塞排查、存储引擎选择等典型场景的解决方法。

## 关键概念
- innodb_lock_wait_timeout: InnoDB 锁等待超时时间，默认值很短
- 死锁日志: 记录死锁发生时的事务信息和锁持有情况
- performance_schema: MySQL 性能监控表，可查询锁等待和线程状态
- MyISAM 表锁: MyISAM 引擎执行 DML 时加表锁，易形成阻塞
- iowait: 操作系统 IO 等待指标，反映磁盘 IO 瓶颈程度

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 笔记中使用了 performance_schema 查询锁等待和线程状态
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 涉及 IO 瓶颈导致 SQL 卡顿的刷盘问题，与本笔记 IO 瓶颈场景相关
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 涉及 InnoDB 刷脏页机制，与锁和 IO 问题有一定关联
