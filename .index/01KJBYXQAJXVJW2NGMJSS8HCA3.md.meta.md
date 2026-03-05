---
note: 01KJBYXQAJXVJW2NGMJSS8HCA3.md
title: 20230128 - 农行crash信息整理并移交
indexed_at: 2026-03-05T08:20:22.604980+00:00
---

## 标签
MySQL, coredump, gdb, 崩溃分析，农行，SQL 调试

## 摘要
记录 2023 年 1 月 5 日和 1 月 9 日两次 MySQL 崩溃事件的 coredump 分析过程，包括调试环境配置、THD 地址获取和问题 SQL 提取。涉及复杂 SQL 查询中 SUBSTRING 和 CASE 语句导致的崩溃问题。

## 关键概念
- THD: MySQL 线程处理对象，存储会话状态和查询信息
- coredump: 程序崩溃时的内存快照，用于事后调试分析
- gdb: GNU 调试器，用于分析 coredump 文件获取崩溃现场信息

## 关联笔记
- 01KJBYXQRY6V6K04B14ER6EQ02.md: 同一农行 crash 事件的原始记录（20230109）
- 01KJBYXQZ8AJVT5AG419NGRPC7.md: 农行 crash 分析整理（20230111）
- 01KJBYXQQS6MFV2NVNDXSN404C.md: 农行 crash 分析整理续（20230119）
