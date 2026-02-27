---
note: 01KJBYD9SKSCBFSFV4QPZYZ5RX.md
title: 20210407 - 同样的压力, 不同机器perf呈现的热点不同
indexed_at: 2026-02-27T10:00:47.852735+00:00
---

用户要求我分析一篇关于 MySQL 性能调试的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 标题：20210407 - 同样的压力，不同机器 perf 呈现的热点不同
2. 内容主要是：
   - 现象：有图片（已截断）
   - 通过 gdb 获取 rec_get_offsets_func 的堆栈跟踪
   - 涉及 MySQL InnoDB 引擎的内部函数调用栈

核心内容分析：
- 这是关于 MySQL 性能分析/调试的笔记
- 使用 gdb 分析 InnoDB 内部函数调用栈
- 涉及 rec_get_offsets_func、page_cur_search_with_match_bytes、btr_cur_search 等 InnoDB 内部函数
- 关于索引记录处理和 B+ 树搜索

标签应该包括：MySQL, InnoDB, 性能分析, gdb, 堆栈分析等

摘要：记录在相同压力下不同机器性能热点差异的现象，通过 gdb 追踪 InnoDB 引擎 rec_get_offsets_func 函数的完整调用堆栈，分析 MySQL 内部索引记录处理流程。

关键概念：
- rec_get_offsets_func: InnoDB 获取记录偏移量的函数
- page_cur_search_with_match_bytes: 页面游标搜索函数
- btr_cur_search: B+ 树游标搜索函数


- dict_load_table: 加载数据字典表的函数

关联笔记方面，01KJBYD9RFGR2SQZQX5P6CB2HJ.md 涉及 gdb 调试技术，01KJBYD9S0A3VZWG60QKDTAF49.md 和 01KJBYD9TKEMSF0Z4RF5DHT1QK.md 都涉及 InnoDB 刷脏页机制，与这篇笔记的 InnoDB 内部分析主题相关。
## 标签
MySQL, InnoDB, 性能分析, gdb, 堆栈追踪, 索引

## 摘要
记录相同压力下不同机器性能热点差异的现象。通过 gdb 追踪 InnoDB 引擎 `rec_get_offsets_func` 函数的完整调用堆栈，分析从索引记录处理到 B+ 树搜索的内部流程。

## 关键概念
- rec_get_offsets_func: InnoDB 获取记录偏移量的核心函数
- page_cur_search_with_match_bytes: 页面游标按字节匹配搜索函数
- btr_cur_search_to_nth_level: B+ 树游标搜索到指定层级的函数
- dict_load_table: 加载数据字典表的函数

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 同属 gdb 调试技术相关笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同属 InnoDB 内部机制研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同属 InnoDB 刷脏页与性能研究
