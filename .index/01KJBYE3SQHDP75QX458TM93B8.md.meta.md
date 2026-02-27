---
note: 01KJBYE3SQHDP75QX458TM93B8.md
title: 20210804 - The total number of locks exceeds the lock table size
indexed_at: 2026-02-27T10:08:11.184827+00:00
---

用户希望我分析这篇关于 MySQL 锁表的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：MySQL InnoDB 锁表空间满的问题（DB_LOCK_TABLE_FULL）
2. 核心内容：
   - 判断条件：buf_LRU_buf_pool_running_out 为 TRUE 时，判定为 DB_LOCK_TABLE_FULL
   - lock 结构何时保存在 buffer pool 中
   - 事务分配了多少 buffer pool 的查看方法

3. 标签应该是：MySQL, InnoDB, 锁表，Buffer Pool, 故障排查

4. 关联笔记分析：
   - 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 如何后台不刷脏页，前台会怎么处理 - 涉及 InnoDB 和 buffer pool
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 涉及 MySQL 和 buffer pool
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了 - 涉及内存分析

让我生成简洁的索引元数据。
## 标签
MySQL, InnoDB, 锁表, Buffer Pool, 故障排查, 源码分析

## 摘要
分析 MySQL InnoDB 引擎 "lock table full" 错误的触发条件和判断逻辑。通过源码追踪 lock 结构在 Buffer Pool 中的分配机制，以及如何通过事务信息观测锁结构占用的堆内存大小。

## 关键概念
- DB_LOCK_TABLE_FULL: InnoDB 锁结构耗尽 Buffer Pool 时的错误码
- buf_LRU_buf_pool_running_out: 判断 Buffer Pool 是否即将耗尽的标志
- lock_heap: 事务锁结构分配的堆内存，可通过 heap size 观测占用量
- mem_heap_create_block_func: 分配内存大于半页时从 Buffer Pool 分配的入口函数

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同属 InnoDB Buffer Pool 机制研究，涉及前台后台刷脏策略
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同为 MySQL Buffer Pool 刷脏页相关研究
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同为内存分析类问题，涉及内存占用排查思路
