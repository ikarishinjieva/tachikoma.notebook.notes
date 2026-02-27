---
title: 20210907 - MySQL内存分配的流程
confluence_page_id: 1343718
created_at: 2021-09-07T08:40:52+00:00
updated_at: 2021-09-07T08:40:52+00:00
---

MySQL向分配器申请内存, 有以下几种情况: 

  1. 经过performance_schema, 进行内存分配
     1. malloc/new: 要求分配器进行内存管理
     2. mmap: 要求分配器不进行内存管理, 直接分配内存
  2. 不经过performance_schema直接分配
     1. malloc/new: 要求分配器进行内存管理
     2. mmap: 要求分配器不进行内存管理, 直接分配内存

分配器向操作系统申请内存

  - 默认情况下, tcmalloc 统计的是 MySQL 通过malloc/new 申请的内存
    - 对于 malloc/new 申请的内存, tcmalloc 自己会维持一定余量 (MySQL释放的内存, 不会立刻释放给操作系统)
  - 增加HEAP_PROFILE_MMAP=true 参数后, tcmalloc也将mmap纳入管理
    - 在这种情况下, tcmalloc统计的内存数值 应与 操作系统的虚拟内存数相同

操作系统分配内存, 是虚拟内存, 内存被使用时, 才真正被使用

  - 相关实验: <https://www.cnblogs.com/xudong-bupt/p/8643094.html>
