---
note: 01KJBYXQQS6MFV2NVNDXSN404C.md
title: 20230119 - 农行crash分析整理 (20230111续)
indexed_at: 2026-03-05T08:26:38.208121+00:00
---

## 摘要
2023 年 1 月 19 日对农行系统 MySQL 崩溃事件的后续分析，基于 1 月 5 日的 coredump 文件进行调试。通过 GDB 获取崩溃线程地址和完整问题 SQL，该 SQL 包含复杂的多表联查和 CASE 语句，涉及 DT_ANSWERS、DT_QUESTIONS、DT_BLOG_META 等表。

## 关键概念
- THD: MySQL 线程处理对象，存储查询字符串和执行上下文
- coredump: 程序崩溃时的内存快照，用于事后调试分析
- GDB: GNU 调试器，用于分析 coredump 获取崩溃现场信息
- m_query_string: THD 结构体成员，保存导致崩溃的原始 SQL 语句

## 关联笔记
- 01KJBYXQZ8AJVT5AG419NGRPC7.md: 前继笔记，20230111 的农行 crash 分析整理
- 01KJBYXQRY6V6K04B14ER6EQ02.md: 20230109 的农行 crash 初始记录，包含工单 BEIJ-3252 和 BEIJ-3260
- 01KJBYXQAJXVJW2NGMJSS8HCA3.md: 20230128 的农行 crash 信息整理并移交，后续跟进
