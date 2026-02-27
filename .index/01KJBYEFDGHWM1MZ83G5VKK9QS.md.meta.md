---
note: 01KJBYEFDGHWM1MZ83G5VKK9QS.md
title: 20210821 - MySQL double write buffer 相关
indexed_at: 2026-02-27T10:09:30.238530+00:00
---

用户需要我分析一篇关于 MySQL double write buffer 的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**: MySQL InnoDB 的 double write buffer 机制
2. **核心内容**: 
   - 参考了3个链接（cnblogs、华为云、MySQL官方文档）
   - 介绍了 `innodb_doublewrite_files` 参数，默认值为2
   - 为每个 buffer pool instance 生成两个 double write file:
     - flush list doublewrite file
     - LRU list doublewrite file
   - 提到了 `innodb_doublewrite_pages` 配置

3. **标签**: 应该包含 MySQL、InnoDB、double write buffer、刷脏页、buffer pool 等关键词

4. **关联笔记**: 从列表中找相关的
   - 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理 - 与刷脏页相关
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 - 直接相关
   - 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表... - 不太相关
   - 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究 - 不相关
   
   最相关的是刷脏页相关的两篇笔记。

5. **关键概念**:
   - innodb_doublewrite_files: 控制每个 buffer pool instance 的 double write file 数量
   - flush list doublewrite file: 用于 flush list 的 double write 文件
   - LRU list doublewrite file: 用于 LRU list 的 double write 文件
   - innodb_doublewrite_pages: 配置 doublewrite 页数的参数

现在生成输出。## 标签
MySQL, InnoDB, double write buffer, 刷脏页, buffer pool, 存储引擎

## 摘要
介绍 MySQL InnoDB 的 double write buffer 机制及相关配置参数。重点说明 `innodb_doublewrite_files` 参数为每个 buffer pool instance 生成两个 double write file（flush list 和 LRU list）的机制。

## 关键概念
- innodb_doublewrite_files: 控制每个 buffer pool instance 生成的 double write file 数量，默认值为 2
- flush list doublewrite file: 用于管理 flush list 的 double write 文件
- LRU list doublewrite file: 用于管理 LRU list 的 double write 文件
- innodb_doublewrite_pages: 配置 doublewrite 页数的参数，默认值与 innodb_write_io_threads 相关

## 关联笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同为 MySQL 刷脏页机制研究，与 double write buffer 目的高度相关
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 探讨 InnoDB 后台刷脏页机制，与 double write buffer 的刷盘场景相关
