---
note: 01KJBYEM2FNRZBZA6CDYKTGYH3.md
title: 20210902 - collation 不同时, _bin collation也可以进行匹配
indexed_at: 2026-02-27T10:10:16.977874+00:00
---

用户希望我分析这篇关于 MySQL collation 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：MySQL 中不同 collation 对索引使用的影响
2. 核心发现：当表校验集为 utf8mb4_general_ci 时，与 utf8mb4_bin 的字符串比较可以使用索引 range，但与其他字符集比较时不能使用索引
3. 涉及的技术概念：MySQL collation、索引、optimizer trace、utf8mb4 字符集

现在我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从给定的列表中选择相关的）

看关联笔记列表，大部分是关于 Clickhouse、内存、刷盘等主题，与 MySQL collation 和索引相关的似乎没有直接关联的。可能需要选择一些 MySQL 相关的，但看起来没有特别匹配的。

让我生成输出：## 标签
MySQL, Collation, 索引优化, utf8mb4, 查询性能, optimizer trace

## 摘要
笔记分析了 MySQL 表 collation 与查询字符串 collation 不同时对索引使用的影响。发现当表校验集为 utf8mb4_general_ci 时，与 utf8mb4_bin 比较可使用索引 range，而与其他 collation 比较则无法使用索引。

## 关键概念
- Collation: 字符集的排序和比较规则，影响字符串匹配行为
- 索引 Range: 索引扫描类型之一，表示使用索引进行范围查找
- Optimizer Trace: MySQL 优化器追踪工具，用于分析查询执行计划的生成过程
- utf8mb4_bin: 二进制排序规则，区分大小写和重音符号

## 关联笔记
无
