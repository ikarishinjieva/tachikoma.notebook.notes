---
note: 01KJBYXQRY6V6K04B14ER6EQ02.md
title: 20230109 - 农行crash
indexed_at: 2026-03-05T08:27:12.260509+00:00
---

## 标签
MySQL, Crash 分析，ARM, temptable, coredump, 农行

## 摘要
记录农行 MySQL 在 ARM 服务器上两次崩溃问题（BEIJ-3252、BEIJ-3260）的初步分析，涉及 temptable 引擎和 stl_vector 断言失败。通过 coredump 获取故障堆栈，确认崩溃位置在 temptable::Handler 相关函数。

## 关键概念
- temptable: MySQL 8.0 内部临时表默认引擎，内存超限时有已知 bug
- stl_vector: C++ 标准库容器，崩溃源于其 operator[] 的越界断言检查
- coredump: 程序崩溃时生成的内存转储文件，用于故障堆栈分析

## 关联笔记
- 01KJBYXQZ8AJVT5AG419NGRPC7.md: 20230111 的后续分析整理，深入分析 1 月 5 日和 1 月 9 日两份 coredump
- 01KJBYXQQS6MFV2NVNDXSN404C.md: 20230119 的续分析，继续排查崩溃原因
- 01KJBYXQAJXVJW2NGMJSS8HCA3.md: 20230128 的信息整理与移交文档
