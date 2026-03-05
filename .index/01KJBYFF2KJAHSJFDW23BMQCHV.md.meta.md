---
note: 01KJBYFF2KJAHSJFDW23BMQCHV.md
title: 20211125 - 滨银汽车MySQL crash
indexed_at: 2026-03-05T07:52:54.757850+00:00
---

## 摘要
记录滨银汽车 MySQL 8.0.11 崩溃问题的分析过程，通过 stacktrace 定位到 Copy_field::invoke_do_copy2 代码位置，判断为内存泄漏导致 m_from_field 变为非法地址。建议开启 coredump 和 general log 功能进一步排查崩溃原因。

## 关键概念
- Copy_field: MySQL 中负责字段数据复制的内部结构体
- check_and_set_temporary_null: 检查并设置临时空值的内部函数
- stacktrace: 程序崩溃时的函数调用栈追踪信息
- coredump: 进程崩溃时生成的内存转储文件，用于事后分析
- 内存泄漏: 内存被错误覆盖或污染导致指针指向非法地址

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同为 MySQL crash coredump 分析案例，涉及 signal 11 崩溃排查
