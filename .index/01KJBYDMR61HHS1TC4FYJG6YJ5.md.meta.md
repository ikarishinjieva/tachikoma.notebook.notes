---
note: 01KJBYDMR61HHS1TC4FYJG6YJ5.md
title: 20210628 - Update 是如何进行的
indexed_at: 2026-02-27T10:05:13.298214+00:00
---

用户要求我分析这篇关于 MySQL UPDATE 语句执行过程的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题是 "20210628 - Update 是如何进行的"
2. 内容主要是 MySQL 5.7.27 源码分析，包括：
   - UPDATE 语句的入口调用堆栈
   - 代码路径整理（Sql_cmd_update::execute 的完整流程）
   - read_record 的调用堆栈分析

3. 关联笔记需要从列表中找出相关的，我看到有几篇可能相关：
   - 01KJBYDKM74PTS0GRF808YHQR7.md: 20210628 - DDL 过程梳理（同一天，都是 MySQL 源码分析）
   - 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 20210628 - Adaptive hash index 代码解析（同一天，都是 MySQL 源码分析）

现在我来生成元数据：

## 标签
MySQL, 源码分析, UPDATE 语句，执行流程，InnoDB

## 摘要
这篇笔记分析了 MySQL 5.7.27 中 UPDATE 语句的完整执行流程，从 Sql_cmd_update::execute 入口到引擎层 ha_update_row 的调用链路。重点梳理了单表 update 的代码路径，包括预处理、条件优化、记录读取和更新等关键阶段。

## 关键概念
- Sql_cmd_update::execute: UPDATE 语句的入口执行函数


- mysql_update: 单表更新的核心处理函数
- read_record: 从存储引擎读取记录的方法，包括 rr_quick 等多种策略
- ha_update_row: 存储引擎层的行更新接口
- Filesort: 当更新涉及索引修改或排序时的处理机制

## 关联笔记
- 01KJBYDKM74PTS0GRF808YHQR7.md: 同一天记录的 MySQL DDL 过程梳理，属于同一系列的源码分析笔记
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 同一天完成的 Adaptive hash index 代码解析，同样聚焦于 MySQL 内部机制## 标签
MySQL, 源码分析, UPDATE 语句，执行流程，InnoDB

## 摘要
这篇笔记分析了 MySQL 5.7.27 中 UPDATE 语句的完整执行流程，从 Sql_cmd_update::execute 入口到引擎层 ha_update_row 的调用链路。重点梳理了单表 update 的代码路径，包括预处理、条件优化、记录读取和更新等关键阶段。

## 关键概念
- Sql_cmd_update::execute: UPDATE 语句的入口执行函数
- mysql_update: 单表更新的核心处理函数
- read_record: 从存储引擎读取记录的方法（如 rr_quick）
- ha_update_row: 存储引擎层的行更新接口
- filesort: 当更新涉及索引修改或排序时的处理机制

## 关联笔记
- 01KJBYDKM74PTS0GRF808YHQR7.md: 同一天记录的 MySQL DDL 过程梳理，属于同一系列源码分析笔记
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 同一天记录的 Adaptive hash index 代码解析，同为 MySQL 内核分析
