---
note: 01KJBYWNT1P6YYR65R1PX640NB.md
title: 20221221 - clickhouse 获取堆栈
indexed_at: 2026-03-05T08:18:43.024300+00:00
---

## 摘要
记录 ClickHouse 中获取和解析线程堆栈的 SQL 方法。通过 `stack_trace` 系统表配合 `addressToSymbol` 和 `demangle` 函数将地址转换为可读的函数名。

## 关键概念
- stack_trace: ClickHouse 系统表，存储线程堆栈信息
- addressToSymbol: 将内存地址转换为符号名的函数
- demangle: 将 C++ 修饰名还原为可读函数名的函数
- thread_number: 线程标识符，用于区分不同线程的堆栈

## 关联笔记
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: 包含 ClickHouse 堆栈分析案例，涉及 AST is too big 错误的堆栈追踪
- 01KJBYQSJVES1CY4G5GA5WTKYF.md: ClickHouse mutations 机制分析，同属 ClickHouse 内部诊断主题
