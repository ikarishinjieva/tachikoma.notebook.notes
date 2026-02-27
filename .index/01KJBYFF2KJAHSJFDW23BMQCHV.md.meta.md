---
note: 01KJBYFF2KJAHSJFDW23BMQCHV.md
title: 20211125 - 滨银汽车MySQL crash
indexed_at: 2026-02-27T10:14:41.736963+00:00
---

用户要求我分析这篇关于 MySQL crash 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于滨银汽车 MySQL 崩溃问题的分析笔记
2. 包含问题描述（MySQL 8.0.11 崩溃堆栈）
3. 分析部分涉及代码级别的调试分析
4. 给出了建议和进展

我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从提供的列表中找相关的）

看关联笔记列表，找与 MySQL、崩溃分析、debug 相关的：
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: gdb 相关 - 可能相关，因为笔记中用到了 gdb 分析
- 01KJBYD9T2SKQ815RG0XWSD236.md: 农行 JDK 导致 mysql connector 报错 - MySQL 相关
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - MySQL 相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - MySQL 相关
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - MySQL 相关


- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 临时磁盘表相关 - 笔记中提到了临时表
- 01KJBYDBWTJGYAJWBZ93943E5R.md: Clickhouse Join - 不太相关
- 01KJBYDD09MGYR4GA0C4KB7P9F.md: DDL 过程梳理 - MySQL 相关
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: Update 是如何进行的 - MySQL 相关

这篇笔记主要涉及 MySQL 崩溃调试和代码分析，最相关的应该是 gdb 调试、performance_schema 监控、以及临时表相关的笔记。

01KJBYD9RFGR2SQZQX5P6CB2HJ.md 关于 gdb 断点的内容与崩溃分析直接相关，01KJBYDAZJTQDJ7X3BQ52XVSSR.md 涉及临时表问题也与笔记中的临时表讨论有联系。
## 标签
MySQL 崩溃，堆栈分析，内存泄漏，MySQL 8.0，调试分析，coredump

## 摘要
分析滨银汽车 MySQL 8.0.11 崩溃问题，通过堆栈追踪定位到 field.h 第 852 行的 Copy_field 调用异常。判断为 m_from_field 指针被内存泄漏污染导致非法地址访问，建议开启 coredump 和 general log 进一步排查。

## 关键概念
- Copy_field: MySQL 查询执行中负责字段拷贝的结构体
- m_is_tmp_nullable: 临时字段是否可为空的标志位
- coredump: 程序崩溃时的内存快照，用于事后分析
- MTS (Multi-threaded slave): MySQL 多线程复制机制

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 都涉及 gdb 调试分析技术
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 都涉及 MySQL 临时表相关问题
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 都涉及 MySQL 问题排查与性能分析
