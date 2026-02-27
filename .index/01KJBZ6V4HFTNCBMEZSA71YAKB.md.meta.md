---
note: 01KJBZ6V4HFTNCBMEZSA71YAKB.md
title: 20240228 - 对embedding进行微调 - 增加训练数据
indexed_at: 2026-02-27T10:47:22.856249+00:00
---

用户要求我分析这篇关于 MySQL 故障案例集合的笔记，生成索引元数据。让我分析内容：

1. 笔记标题：20240228 - 对 embedding 进行微调 - 增加训练数据
2. 内容：主要是 MySQL 各种故障/问题案例的列表，用于训练 embedding 模型的数据

需要输出：
- 标签（3-6 个）
- 摘要（2-3 句话）
- 关键概念（3-5 个）
- 关联笔记（0-3 篇）

从内容看，这是 MySQL 故障案例集合，用于 embedding 训练数据。

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 刷脏页相关 - 有关联（InnoDB）
- 01KJBYD9T2SKQ815RG0XWSD236.md: MySQL connector 报错 - 有关联（MySQL 故障）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema - 有关联（MySQL 诊断）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 有关联
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了 - 可能关联（OOM 问题）
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: Clickhouse MaterializedMySQL - 弱关联


- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 从 binlog 中分离 SQL - 有关联（复制/故障排查）

选择最相关的 3 篇：
1. 01KJBYD9T2SKQ815RG0XWSD236.md - MySQL connector 报错（故障类）
2. 01KJBYD9TKEMSF0Z4RF5DHT1QK.md - MySQL 刷脏页研究（InnoDB 故障相关）
3. 01KJBYD9T7VWC3J9T1JMVWGWMB.md - performance_schema（诊断工具）
## 标签
MySQL, 故障案例, 主从复制, InnoDB, 性能排查, 训练数据

## 摘要
该笔记整理了大量 MySQL 常见故障与问题案例，涵盖主从复制、锁等待、连接异常、OOM、崩溃等场景。这些数据用于对 embedding 模型进行微调训练，以提升故障诊断和检索能力。

## 关键概念
- 主从复制: MySQL 数据同步机制，涉及 IO 线程和 SQL 线程
- 锁等待超时: 事务因等待锁资源超过阈值而失败
- InnoDB: MySQL 默认存储引擎，支持事务和行级锁
- GTID: 全局事务标识符，用于追踪复制状态
- performance_schema: MySQL 性能监控和诊断工具

## 关联笔记
- 01KJBYD9T2SKQ815RG0XWSD236.md: 农行 JDK 导致 mysql connector 报错，同属 MySQL 故障类笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究，涉及 InnoDB 引擎内部机制
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用，可用于故障排查
