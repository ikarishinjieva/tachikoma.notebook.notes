---
note: 01KJBZARNH5DADCGZBKQAHK6PD.md
title: 20240722 - MySQL crash coredump 分析
indexed_at: 2026-02-27T11:07:42.414131+00:00
---

用户要求我分析一篇关于 MySQL crash coredump 分析的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **内容分析**：
   - MySQL 崩溃，signal 11（段错误）
   - GDB 回溯分析
   - 涉及 InnoDB 缓冲池、B+ 树游标、文件排序等
   - MySQL 5.7.31 版本

2. **标签**：需要 3-6 个关键词
   - MySQL, coredump, InnoDB, 崩溃分析, signal 11, 文件排序

3. **摘要**：2-3 句话概括
   - 用户现场 MySQL 发生 signal 11 崩溃，通过 GDB 分析 coredump
   - 崩溃发生在 InnoDB 的缓冲池和 B+ 树操作期间，涉及文件排序过程

4. **关键概念**：3-5 个
   - signal 11: 段错误信号，表示非法内存访问
   - buf_page_optimistic_get: InnoDB 缓冲页乐观获取函数
   - btr_pcur: B+ 树持久游标，用于遍历索引
   - filesort: MySQL 文件排序操作

5. **关联笔记**：从列表中寻找相关的
   - 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页研究 - 涉及 InnoDB
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 涉及 InnoDB 刷脏
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了 - 涉及内存分析

这些笔记都围绕 InnoDB 的内存管理和刷脏机制，与当前崩溃分析的核心问题相关联。## 标签
MySQL, coredump, InnoDB, 崩溃分析, signal 11, filesort

## 摘要
用户现场 MySQL 5.7.31 发生 signal 11 崩溃，通过 GDB 分析 coredump 定位问题。崩溃发生在 InnoDB 缓冲页获取和 B+ 树游标恢复期间，调用链显示正在执行文件排序操作。

## 关键概念
- signal 11: 段错误信号，表示进程尝试访问非法内存地址
- buf_page_optimistic_get: InnoDB 缓冲池乐观获取页的函数，尝试无锁获取缓冲页
- btr_pcur: B+ 树持久游标，用于在索引遍历中保持位置
- filesort: MySQL 当无法使用索引排序时，使用临时文件进行排序的操作

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 涉及 InnoDB 刷脏页机制，与缓冲池管理相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究，同属 InnoDB 缓冲池相关主题
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存分析主题，与崩溃时的内存状态排查相关
