---
note: 01KJBYD9SKSCBFSFV4QPZYZ5RX.md
title: 20210407 - 同样的压力, 不同机器perf呈现的热点不同
indexed_at: 2026-03-05T07:18:53.705828+00:00
---

## 标签
MySQL, InnoDB, 性能分析, 堆栈跟踪, 热点函数, GDB

## 摘要
记录同一压力场景下不同机器性能热点差异的现象。通过 GDB 堆栈分析定位到 `rec_get_offsets_func` 为热点函数，涉及 InnoDB 索引搜索与记录偏移量计算路径。

## 关键概念
- rec_get_offsets_func: InnoDB 中计算记录偏移量的核心函数，用于解析记录格式
- page_cur_search_with_match_bytes: 页面游标搜索函数，执行 B+ 树索引查找
- btr_cur_search_to_nth_level: B+ 树游标搜索到指定层级的核心函数
- dict_load_table: InnoDB 字典加载表的函数，触发索引搜索操作

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同为 MySQL InnoDB 堆栈分析，涉及 btr_cur 和 row_search 等相似调用路径
- 01KJBZEE9JTH4RKQG664GFW88H.md: MySQL crash 分析，涉及 InnoDB 监控和 srv_innodb_monitor_mutex 等底层机制
