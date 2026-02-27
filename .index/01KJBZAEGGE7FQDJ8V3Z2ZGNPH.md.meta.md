---
note: 01KJBZAEGGE7FQDJ8V3Z2ZGNPH.md
title: 20240702 - ChatDBA: 测试两种数据库的工单混合后的相互影响
indexed_at: 2026-02-27T11:04:24.547168+00:00
---

用户希望我分析这篇笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题是关于测试两种数据库（MySQL 和 PostgreSQL）的工单混合后的相互影响
2. 内容是关于一个测试场景：用 MySQL 的参考材料，测试对 PostgreSQL 问题的排查计划是否会有"错误的"影响
3. 笔记中包含了一个提示词模板，用于测试 AI DBA 专家系统
4. 还有一份生成的 MySQL 死锁排查文档

核心内容：
- 测试跨数据库知识干扰（MySQL 知识对 PG 问题排查的影响）
- 死锁排查方法
- 排查计划的伪代码格式规范

关键概念：
- 死锁（Deadlock）
- ShareLock
- 排查计划/诊断流程
- 工单匹配度评估
- 增量修改

关联笔记：
从列表中看，以下笔记可能相关：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 如何后台不刷脏页（MySQL 相关）
- 01KJBYD9T2SKQ815RG0XWSD236.md: 农行 JDK 导致 mysql connector 报错（MySQL 相关）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用（MySQL 相关）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究


- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究

这些笔记都涉及 MySQL 数据库的内部机制和性能问题，与当前测试的跨数据库知识干扰场景相关。
## 标签
数据库死锁，MySQL, PostgreSQL, 排查计划，AI 测试，工单匹配

## 摘要
本笔记记录了一个测试场景：验证 MySQL 的参考材料是否会对 PostgreSQL 死锁问题的排查计划产生错误影响。包含完整的 AI 提示词模板、排查计划伪代码格式规范，以及一份生成的 MySQL 死锁排查文档。

## 关键概念
- ShareLock 死锁：PostgreSQL 中由于并发事务间资源竞争导致的锁定冲突
- 排查计划伪代码：使用 SWITCH/Case/THINKING/FIND/IF/RETURN 等函数描述诊断流程的规范格式
- 工单匹配度：评估参考文档与当前问题的相关程度，分为完全匹配/较高/较低/无关四级
- 增量修改：使用*_SPECULATIVE 系列函数对排查计划进行补丁式修改的方法

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 涉及 InnoDB 刷脏页机制，同属 MySQL 内部原理研究
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 使用，与 MySQL 性能诊断相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究，同属数据库底层机制分析
