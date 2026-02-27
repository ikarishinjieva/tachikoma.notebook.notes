---
title: 20210804 - The total number of locks exceeds the lock table size
confluence_page_id: 1343574
created_at: 2021-08-04T09:54:14+00:00
updated_at: 2021-08-04T14:20:41+00:00
---

# 判断条件

在源码 mysql-8.0.18/storage/innobase/include/db0err.h 找到如下说明: 

```
  /** lock structs have exhausted the buffer pool (for big transactions,
  InnoDB stores the lock structs in the buffer pool) */
  DB_LOCK_TABLE_FULL,
``` 

判断条件是 buf_LRU_buf_pool_running_out 为TRUE时, 判 DB_LOCK_TABLE_FULL

![image2021-8-4 22:15:13.png](/assets/01KJBYE3SQHDP75QX458TM93B8/image2021-8-4%2022%3A15%3A13.png)

# lock 结构什么时候保存在buffer pool中

线索: mem_heap_create_block_func, 在分配数 > 半页, 会调用: buf_block_alloc, 从buffer pool中分配内存

# 事务分配了多少buffer pool

```
Total number of lock structs in row lock hash table 440
LIST OF TRANSACTIONS FOR EACH SESSION:
—TRANSACTION 0 42306999, ACTIVE 133 sec, process no 10099, OS thread id 1878960
441 lock struct(s), heap size 44352
``` 

heap size 是 mem_heap_get_size(trx->lock.lock_heap)

![image2021-8-4 17:59:54.png](/assets/01KJBYE3SQHDP75QX458TM93B8/image2021-8-4%2017%3A59%3A54.png)

![image2021-8-4 21:21:50.png](/assets/01KJBYE3SQHDP75QX458TM93B8/image2021-8-4%2021%3A21%3A50.png)

![image2021-8-4 21:22:27.png](/assets/01KJBYE3SQHDP75QX458TM93B8/image2021-8-4%2021%3A22%3A27.png)
