---
note: 01KJBYF3TWWJXVBNENSFEAPQD5.md
title: 20211029 - MySQL crash 分析
indexed_at: 2026-03-05T07:48:22.303991+00:00
---

## 标签
MySQL, 崩溃分析, InnoDB, coredump, gdb, signal 11

## 摘要
记录 MySQL 8.0.23 企业版在执行 `select * from bus_deposit_info` 查询时发生 signal 11 崩溃的完整分析过程。通过 coredump 和 gdb 堆栈分析，定位到 InnoDB 引擎的 `ha_innobase::index_read` 函数调用链中的崩溃点。

## 关键概念
- signal 11: Unix 系统中的段错误信号，通常由非法内存访问触发
- ha_innobase: MySQL InnoDB 存储引擎的处理器接口类
- coredump: 程序崩溃时的内存快照，用于事后调试分析
- gdb: GNU 调试器，用于分析程序崩溃堆栈
- InnoDB: MySQL 默认的事务型存储引擎

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同样是 MySQL crash coredump 分析，涉及 signal 11 和 InnoDB 堆栈调试
- 01KJBYXQNR1PE2DGTP93ETQCDJ.md: MySQL 8.0 升级后崩溃分析，使用 coredump 诊断
