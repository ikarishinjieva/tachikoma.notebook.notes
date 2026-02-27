---
note: 01KJBYDRCGKBD0D05TYP2MN78X.md
title: 20210715 - 一个insert的p_s输出
indexed_at: 2026-02-27T10:07:18.969784+00:00
---

用户要求我分析一篇关于 MySQL INSERT 语句的 performance_schema 分析笔记，并生成索引元数据。让我先理解笔记内容：

1. 这是一篇关于 MySQL 单机 INSERT 操作的 performance_schema 分析笔记
2. 包含了 statement、stage、wait 三个部分的 performance_schema 表查询结果
3. 主要展示了 INSERT 语句的执行过程和各个阶段的性能数据

根据内容，我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个核心概念
4. 关联笔记：从提供的笔记列表中找出 0-3 篇相关的

关联笔记分析：
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 → 直接相关，都是 performance_schema
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 20210628 - Update 是如何进行的 → 相关，都是 SQL 语句执行分析
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: 20210628 - DDL 过程梳理 → 相关，都是 SQL 执行过程分析

让我生成输出。
## 标签
MySQL, performance_schema, INSERT, 性能分析, 执行阶段, 等待事件

## 摘要
记录了单机 INSERT 语句在 performance_schema 中的完整执行轨迹，包括 statement 维度的统计信息和 stage 执行阶段。通过 events_statements/stages/waits_history_long 三张表展示了 SQL 从开始到清理的全过程。

## 关键概念
- events_statements_history_long: 记录 SQL 语句级别的执行统计，包括耗时、锁时间、影响行数等
- events_stages_history_long: 记录 SQL 执行的各个阶段，如 opening tables、update、closing tables 等
- events_waits_history_long: 记录线程的等待事件，包括 IO、锁、空闲等等待类型

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同属 performance_schema 使用系列，该笔记介绍日常使用方法
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: 相似的 SQL 执行过程分析，该笔记分析 UPDATE 语句的执行机制
