---
note: 01KJBYE2S5BW09A60PY13ARNZM.md
title: 20210730 - i_s.columns 排序不一致
indexed_at: 2026-03-05T07:38:52.489053+00:00
---

## 标签
MySQL, information_schema, 排序问题, lower_case_table_names, 内部表，视图

## 摘要
记录 MySQL 8.0.19 中 information_schema.COLUMNS 视图查询结果排序不一致的现象，不同环境下即使 lower_case_table_names 参数相同，输出顺序也可能按 ORDINAL_POSITION 或 TABLE_NAME 排序。通过堆栈分析推测排序发生在数据存入 mysql.columns 内部表的过程中。

## 关键概念
- information_schema.COLUMNS: MySQL 元数据视图，展示所有表的列信息
- lower_case_table_names: 控制表名大小写敏感的 MySQL 配置参数
- mysql.columns: 存储列元数据的 InnoDB 内部表
- ORDINAL_POSITION: 列在表定义中的顺序位置
- NestedLoopIterator: MySQL 查询执行引擎中的嵌套循环迭代器

## 关联笔记
- 01KJBZAND9YASRASV3Z9BGA4MA.md: 涉及 lower_case_table_names 参数导致表名大小写敏感问题的排查案例
