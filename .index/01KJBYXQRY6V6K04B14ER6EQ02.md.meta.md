---
note: 01KJBYXQRY6V6K04B14ER6EQ02.md
title: 20230109 - 农行crash
indexed_at: 2026-02-27T10:24:28.913984+00:00
---

用户要求我分析这篇关于农行 MySQL 崩溃问题的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：农行 MySQL 崩溃问题分析（crash）
2. **核心内容**：
   - 测试环境信息（ARM 服务器）
   - 两次崩溃事件（BEIJ-3252, BEIJ-3260）
   - 崩溃堆栈分析（temptable::Handler, stl_vector.h:805）
   - Coredump 分析
   - 涉及 MySQL 8.0.18 temptable 组件

3. **关键概念**：
   - temptable::Handler - MySQL 临时表处理器
   - stl_vector - C++ STL 向量容器
   - Coredump - 程序崩溃转储文件
   - ARM 服务器
   - MySQL 8.0.18

4. **关联笔记判断**：
   查看其他笔记列表，寻找与 MySQL、崩溃分析、临时表、performance_schema 等相关的笔记：
   - 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（农行 + MySQL 相关）
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（MySQL 调试相关）
   - 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习（MySQL 相关但主题是 Clickhouse）
   - 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表相关（与 temptable 临时表主题相关）

最相关的应该是农行相关的 MySQL 问题笔记，以及临时表相关的笔记。

我需要用中文输出，保持 Markdown 格式。
## 标签
MySQL 崩溃, Coredump 分析, temptable, ARM 服务器, 堆栈调试, 农行

## 摘要
记录农行测试环境 ARM 服务器上 MySQL 8.0.18 两次崩溃问题的排查过程。通过 Coredump 分析定位到 temptable::Handler 组件中 stl_vector 取数操作符越界断言失败导致崩溃。

## 关键概念
- temptable::Handler: MySQL 临时表存储引擎处理器
- stl_vector: C++ STL 向量容器，崩溃位置在取数操作符 [] 的断言检查
- Coredump: 程序崩溃时生成的内存转储文件，用于故障堆栈分析
- ARM 服务器: 崩溃发生的目标硬件平台

## 关联笔记
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同为农行 MySQL 相关问题（JDK 导致 connector 报错）
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 同样涉及 MySQL 临时磁盘表问题分析
