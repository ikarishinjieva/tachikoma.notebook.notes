---
note: 01KJBYKCTPB0T5X02RSRVRS3W4.md
title: 20220628 - MySQL crash (bpage->buf_fix_count == 0)
indexed_at: 2026-03-05T08:04:30.500058+00:00
---

## 标签
MySQL, InnoDB, crash, 断言失败, buf_fix_count, LRU

## 摘要
记录 MySQL 8.0.20 生产环境 InnoDB 崩溃问题，断言失败 `bpage->buf_fix_count == 0` 位于 buf0lru.cc:1997。分析指出崩溃发生在 buf_LRU_free_page 过程中 buf_fix_count 从 0 变为非 0，可能是非法更新或锁竞争导致。

## 关键概念
- buf_fix_count: InnoDB 缓冲页的引用计数器，记录当前有多少线程正在使用该页
- buf_LRU_free_page: InnoDB LRU 算法中释放缓冲页的函数，要求页的引用计数为 0
- 断言失败: InnoDB 内部一致性检查失败，通常表明存在竞态条件或内存损坏
- 缓冲池: InnoDB 用于缓存数据页和索引页的内存区域

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同为 MySQL crash coredump 分析，涉及 buf_page_optimistic_get 函数和类似的 InnoDB 崩溃场景
- 01KJBZ4V4AX7E85YRY3QG1H2W3.md: 博般 MySQL 5.7.31 crash 案例，涉及 btr_cur_optimistic_latch_leaves 和 row_search_mvcc 等相似的 InnoDB 崩溃堆栈
