---
note: 01KJBYE3SQHDP75QX458TM93B8.md
title: 20210804 - The total number of locks exceeds the lock table size
indexed_at: 2026-03-05T07:39:23.036147+00:00
---

## 摘要
分析 InnoDB 报错"The total number of locks exceeds the lock table size"的根本原因：大事务的锁结构体耗尽 buffer pool。通过源码追踪锁结构存储机制、heap size 计算及 buf_LRU_buf_pool_running_out 判断条件。

## 关键概念
- DB_LOCK_TABLE_FULL: InnoDB 错误码，表示锁结构体已耗尽 buffer pool 空间
- buf_LRU_buf_pool_running_out: buffer pool 空间不足的判断标志，触发 DB_LOCK_TABLE_FULL
- mem_heap_create_block_func: 内存分配函数，分配超过半页时调用 buf_block_alloc 从 buffer pool 分配
- lock struct: InnoDB 锁结构体，大事务时存储在 buffer pool 中
- lock_heap: 事务锁堆，通过 mem_heap_get_size(trx->lock.lock_heap) 获取 heap size

## 关联笔记
- 01KJBYFD3950T3XNVP5R4BXYZV.md: 同属 InnoDB 锁分析，包含 lock struct 和 heap size 的类似诊断信息
- 01KJBYDV2NWRKK7EW365CFMZG6.md: 涉及 mem_heap_create_block_func 的内存分配机制分析
