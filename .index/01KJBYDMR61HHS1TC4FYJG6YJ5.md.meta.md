---
note: 01KJBYDMR61HHS1TC4FYJG6YJ5.md
title: 20210628 - Update 是如何进行的
indexed_at: 2026-03-05T07:32:22.199200+00:00
---

## 标签
MySQL, UPDATE 执行流程, sql_update.cc, InnoDB, 行更新, 调用堆栈

## 摘要
分析 MySQL 5.7.27 中 UPDATE 语句的完整执行路径，从 Sql_cmd_update::execute 入口到 InnoDB 引擎层的 ha_update_row。详细梳理单表 update 的代码流程，包括表锁、条件优化、记录读取（init_read_record/rr_quick）和行更新操作。

## 关键概念
- Sql_cmd_update::execute: UPDATE 语句在 Server 层的执行入口，区分单表更新和 multi-table 更新
- init_read_record: 初始化从存储引擎读取记录的方法，从七种读取方式中选择合适的一种
- rr_quick: 基于索引快速扫描的记录读取方法，用于范围查询或等值查询
- row_search_mvcc: InnoDB 引擎中通过 MVCC 机制搜索记录的核心函数
- ha_update_row: 存储引擎层的行更新接口，将修改后的行写回引擎

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 包含 row_search_mvcc 和 join_init_read_record 的调用堆栈，与 read_record 流程相关
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 分析 UPDATE 操作时的 InnoDB 锁竞争堆栈，涉及 row_search_mvcc 和 lock_table
- 01KJBZEE9JTH4RKQG664GFW88H.md: 包含大量 InnoDB 内部堆栈（trx_sys、row_search_mvcc），与引擎层执行相关
