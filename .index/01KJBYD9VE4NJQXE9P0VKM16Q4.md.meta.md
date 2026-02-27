---
note: 01KJBYD9VE4NJQXE9P0VKM16Q4.md
title: 20210329 - MySQL刷盘研究 - 数据文件刷盘慢导致SQL卡顿
indexed_at: 2026-02-27T10:01:33.641804+00:00
---

Let me analyze this note about MySQL flushing research. The note discusses:

1. MySQL InnoDB flushing mechanisms, specifically data file flushing causing SQL stalls
2. Technical details about innodb_data_pending_fsyncs and related metrics
3. A stack trace showing SQL thread blocking when data file flush latency increases to 1s
4. References to InnoDB flushing research

Key concepts to extract:
- innodb_data_pending_fsyncs
- fil_n_pending_log_flushes
- fil_n_pending_tablespace_flushes
- fil_flush
- Data file flushing

Related notes from the list:
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 (Directly related - dirty page flushing research)
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理 (Related - InnoDB background/foreground flushing)
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 (Somewhat related - performance monitoring)

Let me create the index metadata.## 标签
MySQL, InnoDB, 刷盘，fsync, 性能优化，堆栈分析

## 摘要
研究 MySQL InnoDB 数据文件刷盘机制，分析 fil_flush 等关键函数的调用链路。通过 GDB 堆栈追踪 SQL 卡顿根因，验证数据文件 flush 延迟增至 1s 会导致线程阻塞。

## 关键概念
- innodb_data_pending_fsyncs: 待刷盘 fsync 总数，包含日志和表空间两类
- fil_n_pending_log_flushes: FIL_TYPE_LOG 类型的待刷盘数，等于 innodb_os_log_pending_fsyncs
- fil_n_pending_tablespace_flushes: FIL_TYPE_IMPORT/TABLESPACE 类型的待刷盘数
- fil_flush: InnoDB 表空间刷盘核心函数，位于 fil0fil.cc

## 关联笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同系列 MySQL 刷脏页研究，主题高度相关
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 研究 InnoDB 后台/前台刷脏页机制，补充刷盘策略理解
