---
note: 01KJBYXQNR1PE2DGTP93ETQCDJ.md
title: 20230118 - 中移 crash
indexed_at: 2026-03-05T08:25:56.066582+00:00
---

## 标签
MySQL, 崩溃分析, InnoDB, coredump, 版本升级，断言失败

## 摘要
记录 MySQL 8.0.30 从 8.0.27 升级后运行崩溃的故障，通过 mysql-debug 生成 coredump 并分析堆栈。定位到 innobase/rem0rec.cc 中 rec_convert_dtuple_to_rec_new 函数的断言失败（!rec_new_is_versioned(rec)），崩溃发生在 row_undo_mod 事务回滚路径。

## 关键概念
- coredump 分析：通过调试版本生成内存转储文件，用于定位崩溃根因
- InnoDB 断言失败：运行时检查条件不满足触发的保护性崩溃机制
- row_undo_mod：InnoDB 事务回滚时修改聚簇索引记录的核心函数
- 版本化记录：InnoDB MVCC 机制中用于多版本并发控制的记录格式
- btr_cur_pessimistic_update：B+ 树游标悲观更新操作，涉及记录转换和插入

## 关联笔记
- 01KJBYKCTPB0T5X02RSRVRS3W4.md: 同为 MySQL 8.0.x 版本的 assertion failure 崩溃分析案例
