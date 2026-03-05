---
note: 01KJBYHCENW7J7KPJMGXJGAK4S.md
title: 20220302 - 工行MySQL crash
indexed_at: 2026-03-05T07:57:32.533576+00:00
---

## 摘要
记录工行 MySQL 实例崩溃问题的分析过程，包含崩溃堆栈解析和断点复现。怀疑编译优化导致堆栈信息丢失，崩溃原因为 signal 11 内存地址错误。

## 关键概念
- InnoDB: MySQL 的默认存储引擎，负责事务和行级锁定
- page_cur_delete_rec: InnoDB 页面游标删除记录的核心函数
- btr_cur_optimistic_delete: B+ 树游标乐观删除操作
- row_purge: InnoDB 行级数据清理操作
- signal 11: 段错误信号，表示非法内存访问

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同为 MySQL crash 的 coredump 分析，涉及 InnoDB signal 11 错误
