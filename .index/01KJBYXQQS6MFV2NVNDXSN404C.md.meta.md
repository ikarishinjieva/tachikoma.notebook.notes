---
note: 01KJBYXQQS6MFV2NVNDXSN404C.md
title: 20230119 - 农行crash分析整理 (20230111续)
indexed_at: 2026-02-27T10:24:18.155837+00:00
---

用户希望我分析这篇关于农行 crash 分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 MySQL 崩溃分析的技术笔记
2. 涉及 coredump 调试、GDB 分析
3. 有一个复杂的 SQL 查询导致崩溃
4. 与之前 20230111 的笔记有关联

现在我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：从给出的笔记列表中找 0-3 篇相关的

关于关联笔记，我需要找与 MySQL、crash、调试、性能相关的：
- 20210330 - 农行 JDK 导致 mysql connector 报错（同是农行项目）
- 20210330 - performance_schema 的日常使用（MySQL 调试相关）
- 20210404 - MySQL 刷脏页研究（MySQL 相关）
- 20210329 - MySQL 刷盘研究（MySQL 相关）

最相关的是：
1. 20210330 - 农行 JDK 导致 mysql connector 报错 - 同是农行项目的 MySQL 问题
2. 20210330 - performance_schema 的日常使用 - MySQL 调试相关

让我生成输出。
## 标签
MySQL, coredump, GDB 调试, 崩溃分析, 农行, SQL 查询

## 摘要
记录 2023 年 1 月 5 日农行系统 MySQL 崩溃的 coredump 分析过程，通过 GDB 获取崩溃现场的 THD 地址和问题 SQL。涉及一个复杂的多表关联查询，包含多个 CASE WHEN 分支和子查询。

## 关键概念
- THD: MySQL 线程处理对象，存储连接和查询状态
- coredump: 程序崩溃时的内存快照，用于事后调试分析
- m_query_string: THD 结构中保存当前执行 SQL 的字段
- GDB: GNU 调试器，用于分析 coredump 定位崩溃原因

## 关联笔记
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同为农行项目 MySQL 相关问题排查
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 用于 MySQL 性能问题诊断
