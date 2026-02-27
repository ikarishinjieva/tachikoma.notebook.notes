---
note: 01KJBYDKM74PTS0GRF808YHQR7.md
title: 20210628 - DDL 过程梳理
indexed_at: 2026-02-27T10:05:02.174186+00:00
---

用户需要我分析一篇关于 MySQL DDL 过程的笔记，并生成索引元数据。让我仔细阅读笔记内容，提取关键信息。

笔记标题：20210628 - DDL 过程梳理

内容主要是关于 MySQL 8.0 中 ALTER TABLE 的 DDL 流程，包括：
- 入口函数 mysql_alter_table
- mysql_prepare_alter_table
- 创建临时表结构
- inplace/instant 类型的处理流程
- 引擎支持状况检查
- online DDL 的锁处理
- ha_prepare_inplace_alter_table
- ha_inplace_alter_table
- ha_commit_inplace_alter_table

关键概念：
1. inplace DDL - 原地修改表结构，不需要重建表
2. instant DDL -  instant 添加/删除列，几乎不锁表
3. online DDL - 允许并发访问的 DDL 操作
4. 临时表 - DDL 过程中创建的临时结构
5. 聚簇索引重建 - 某些 DDL 操作需要重建聚簇索引

关联笔记：
从列表中看，以下笔记可能有关联：
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: 20210628 - Update 是如何进行的（同一天，都是 MySQL 内部机制）
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 20210628 - Adaptive hash index 代码解析（同一天，都是 MySQL InnoDB 代码解析）


- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理（InnoDB 相关）

最相关的是同一天的两篇 MySQL 代码解析笔记。

标签应该包括：MySQL, DDL, ALTER TABLE, InnoDB, inplace, online DDL

摘要：这篇笔记详细梳理了 MySQL 8.0 中 ALTER TABLE 的 DDL 执行流程，包括 inplace 和 instant 两种优化方式的处理机制。

涵盖了从入口函数到引擎层的完整调用链，以及锁策略和事务处理的关键细节。

关键概念涉及 inplace/instant DDL、online DDL 的锁机制、临时表结构和聚簇索引重建等核心内容。## 标签
MySQL, DDL, ALTER TABLE, InnoDB, inplace, online DDL

## 摘要
详细梳理 MySQL 8.0 中 ALTER TABLE 的 DDL 执行流程，从入口函数 mysql_alter_table 到引擎层的完整调用链。重点分析 inplace/instant 两种优化方式的支持判断、锁策略及事务处理机制。

## 关键概念
- inplace DDL: 原地修改表结构，避免全表重建的优化方式
- instant DDL: 仅修改元数据即可完成的列操作，几乎无锁
- online DDL: 允许并发读写访问的 DDL 执行模式
- 临时表结构: DDL 过程中创建的中间表用于数据重建
- 聚簇索引重建: 某些 DDL 操作需要读取并重建聚簇索引数据

## 关联笔记
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: 20210628 - Update 是如何进行的（同一天记录，都是 MySQL InnoDB 内部操作机制）
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 20210628 - Adaptive hash index 代码解析（同一天记录，都是 InnoDB 代码流程分析）
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理（InnoDB 刷页机制相关）
