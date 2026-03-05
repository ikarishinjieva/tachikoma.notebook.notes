---
note: 01KJBYEFDGHWM1MZ83G5VKK9QS.md
title: 20210821 - MySQL double write buffer 相关
indexed_at: 2026-03-05T07:41:41.701425+00:00
---

## 摘要
介绍 MySQL 8.0 InnoDB 的 double write buffer 机制及相关配置参数 `innodb_doublewrite_files` 和 `innodb_doublewrite_pages`。说明每个 buffer pool instance 生成两个 double write file（flush list 和 LRU list），并详细描述其默认大小计算方式。

## 关键概念
- innodb_doublewrite_files: 每个 buffer pool instance 生成的 double write file 数量，默认值为 2
- flush list doublewrite file: 用于 flush list 的 double write 文件，大小为 InnoDB page size * doublewrite page bytes
- LRU list doublewrite file: 用于 LRU list 的 double write 文件，大小与 doublewrite pages 和 buffer pool instances 数量相关
- innodb_doublewrite_pages: 配置 doublewrite pages 数量，默认值等于 innodb_write_io_threads

## 关联笔记
- 01KJBYED7JBE2YBE7Y4YY213H6.md: 分析 MySQL 内存分配时提到 double write buffer 机制，引用了本篇笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 研究 MySQL 刷脏页机制时提到 doublewrite buffer 竞争可能导致刷盘延迟
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 分析数据文件刷盘问题时涉及刷 double write buffer 的代码路径
