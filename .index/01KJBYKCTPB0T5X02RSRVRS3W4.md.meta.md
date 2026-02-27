---
note: 01KJBYKCTPB0T5X02RSRVRS3W4.md
title: 20220628 - MySQL crash (bpage->buf_fix_count == 0)
indexed_at: 2026-02-27T10:17:29.034191+00:00
---

用户要求我分析一篇关于 MySQL crash 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 MySQL 8.0.20 在生产环境 crash 的问题记录
2. 崩溃原因是 InnoDB 的 assertion failure: buf0lru.cc:1997:bpage->buf_fix_count == 0
3. 有完整的堆栈信息
4. 环境是一主一从架构，OpenEuler 系统

我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

看关联笔记列表，相关的可能是：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理（InnoDB 相关）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究（InnoDB 刷页相关）
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（可能涉及 buffer pool）

这些笔记与 InnoDB buffer pool、刷页机制相关，与当前 crash 的 buf0lru.cc 有关联。## 标签
MySQL, InnoDB, 崩溃分析, Buffer Pool, 断言失败, 生产故障

## 摘要
记录客户生产环境 MySQL 8.0.20 夜间无变更情况下发生的 crash 问题，崩溃原因为 InnoDB 断言失败 `bpage->buf_fix_count == 0`。笔记包含完整的错误日志和调用堆栈，指向 buf0lru.cc 的 LRU 页管理逻辑。

## 关键概念
- buf_fix_count: InnoDB 缓冲页的引用计数，用于跟踪页的使用状态
- buf_LRU_free_page: InnoDB LRU 链表中释放缓冲页的核心函数
- Assertion Failure: 程序运行时断言检查失败，通常表示内部状态不一致

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同涉及 InnoDB 刷脏页与缓冲页管理机制
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同涉及 MySQL 刷脏页与 Buffer Pool 研究
