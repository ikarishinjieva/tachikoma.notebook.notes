---
note: 01KJBYEM2FNRZBZA6CDYKTGYH3.md
title: 20210902 - collation 不同时, _bin collation也可以进行匹配
indexed_at: 2026-03-05T07:43:29.302082+00:00
---

## 摘要
分析 MySQL 中不同 collation 比较时的索引使用行为：当表 collation 为 utf8mb4_general_ci 时，与 utf8mb4_bin 比较可使用索引 range 扫描，而与其他 collation（如 utf8mb4_is_0900_as_cs）比较则全表扫描。通过 optimizer_trace 分析索引选择决策过程。

## 关键概念
- collation: 字符集的排序规则，决定字符串比较和排序的方式
- _bin collation: 二进制排序规则，按字节值直接比较
- 索引 range 扫描: 使用索引的范围扫描访问方式，比全表扫描更高效
- optimizer_trace: MySQL 优化器跟踪工具，用于分析查询优化决策过程
- Using index condition: EXPLAIN 输出中表示使用索引条件下推的标记

## 关联笔记
- 01KJBZEE9JTH4RKQG664GFW88H.md: 同为 MySQL 问题分析笔记，涉及 MySQL 内部机制调试
- 01KJBZ6V4HFTNCBMEZSA71YAKB.md: 包含 MySQL 字符集和索引相关的问答记录
