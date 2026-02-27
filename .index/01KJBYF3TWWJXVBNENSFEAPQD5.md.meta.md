---
note: 01KJBYF3TWWJXVBNENSFEAPQD5.md
title: 20211029 - MySQL crash 分析
indexed_at: 2026-02-27T10:12:58.488596+00:00
---

用户要求我分析这篇关于 MySQL crash 分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 MySQL 崩溃分析的笔记
2. 崩溃时间：2021-10-27 06:39:20
3. MySQL 版本：8.0.23 企业版
4. 崩溃原因：signal 11 (段错误)
5. 崩溃时执行的查询：select * from bus_deposit_info
6. 从堆栈看，崩溃发生在 InnoDB 的 index_read/index_first/rnd_next 调用链中
7. 使用了 gdb 分析 coredump

现在需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：从提供的列表中找 0-3 篇相关的

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页相关 - 有一定关联（都是 InnoDB 相关）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 有一定关联
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 临时磁盘表相关 - 关联度较低
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: Adaptive hash index 代码解析 - 都是 InnoDB 内部机制，有一定关联


- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 关联度较低

最相关的应该是 InnoDB 相关的笔记，特别是涉及刷脏页和内部机制的内容。
## 标签
MySQL 崩溃分析，InnoDB，Signal 11，GDB 调试，堆栈分析，企业版 8.0.23

## 摘要
记录 MySQL 8.0.23 企业版因 Signal 11 崩溃的现场日志与 GDB 调试分析。崩溃发生在执行 `select * from bus_deposit_info` 查询时，堆栈显示问题出在 InnoDB 的 index_read/rnd_next 调用链中。

## 关键概念
- Signal 11: 段错误信号，通常由访问无效内存地址引起
- ha_innobase: InnoDB 存储引擎的 Handler 接口层
- index_read/rnd_next: InnoDB 索引读取和随机遍历的核心函数
- Materialized_cursor: MySQL 物化游标，用于处理子查询或临时结果集

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同属 InnoDB 刷页机制相关研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同属 InnoDB 刷脏页研究方向
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 同属 InnoDB 内部机制代码分析
