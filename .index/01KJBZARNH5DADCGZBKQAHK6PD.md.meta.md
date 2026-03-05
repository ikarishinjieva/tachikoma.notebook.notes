---
note: 01KJBZARNH5DADCGZBKQAHK6PD.md
title: 20240722 - MySQL crash coredump 分析
indexed_at: 2026-03-05T10:22:45.005145+00:00
---

## 摘要
记录 MySQL 5.7.31 在生产现场发生 crash 的 coredump 分析过程，错误为 signal 11。通过 gdb backtrace 定位崩溃发生在 InnoDB 的 buf_page_optimistic_get 函数，调用链涉及 B-Tree 游标恢复和行查询操作。

## 关键概念
- signal 11: SIGSEGV 段错误，通常表示访问了无效的内存地址
- buf_page_optimistic_get: InnoDB 缓冲池页面乐观获取函数，用于读取数据页
- btr_pcur: InnoDB B-Tree 持久化游标，用于在索引树中定位和遍历记录
- filesort: MySQL 排序操作，当无法使用索引排序时将数据写入临时文件排序

## 关联笔记
- 01KJBZAPGEQT0E9NB6C8CFG229.md: 记录 MySQL 执行 select 语句 crash 的排查流程和思维链方法
- 01KJBZAQZ6CX2KZQ4FC02BPFXS.md: 涉及 signal 11 错误相关的知识文档检索和图谱构建
- 01KJBZED41K5CMAYSC5QKC2E76.md: 分析 MySQL InnoDB 的 mutex 锁机制，涉及 signal_count 相关代码
