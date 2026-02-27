---
note: 01KJBYJEQN14KSK24XPHFC6S1M.md
title: 20220528 - page_cleaner日志的模拟方法
indexed_at: 2026-02-27T10:16:44.399195+00:00
---

用户要求我分析一篇关于 MySQL page_cleaner 日志模拟方法的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：研究 MySQL InnoDB 的 page_cleaner 日志连续打印周期，使用 GDB 调试模拟
2. **核心技术**：MySQL 5.7.26/5.7.27, InnoDB, page_cleaner, GDB 调试
3. **方法**：通过 GDB 断点和命令模拟刷盘周期，使每个计算周期触发特定条件
4. **结论**：展示了通过调试得到的 page_cleaner 日志输出

**标签**应该包括：MySQL, InnoDB, page_cleaner, GDB 调试，刷脏页，日志分析

**摘要**：这篇笔记记录了如何使用 GDB 调试 MySQL 5.7 的 page_cleaner 模块，通过设置断点和延迟来模拟刷盘周期，验证日志打印的内部计算逻辑。

**关键概念**：
- page_cleaner: InnoDB 后台刷脏页的协调器线程
- buf_flush_page_cleaner_coordinator: page_cleaner 的协调函数
- 刷盘周期：page_cleaner 计算和执行刷脏页的时间周期
- GDB 调试命令：用于中断和注入延迟的调试技术

**关联笔记**：
从列表中找相关的：
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - performance_schema 的日常使用 - 关联度低


- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 - 高度相关，都是关于刷脏页
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 相关，都是刷盘主题
- 01KJBYD9RN1BMGGWMYS78NG817.md: 20210404 - 如何给 LWP 降速 - 相关，涉及线程调速技术
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb 只停止触发断点的线程 - 相关，都是 GDB 调试技术

最相关的应该是刷脏页研究和 GDB 调试相关的笔记。
## 标签
MySQL, InnoDB, page_cleaner, GDB 调试，刷脏页，日志分析

## 摘要
本笔记记录了使用 GDB 调试 MySQL 5.7 InnoDB 的 page_cleaner 模块，通过设置断点和注入延迟来模拟刷盘周期。验证了 page_cleaner 日志打印的内部计算逻辑，包括时间间隔、警告计数等参数的变化规律。

## 关键概念
- page_cleaner: InnoDB 负责后台协调刷脏页的线程
- buf_flush_page_cleaner_coordinator: page_cleaner 的核心协调函数
- 刷盘周期：page_cleaner 预期 1000ms 的实际执行循环时间
- GDB 断点命令：通过 shell sleep 注入延迟来触发特定代码路径

## 关联笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同为 MySQL 刷脏页主题的研究笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 涉及 GDB 调试中线程停止控制的技术
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同为 MySQL 刷盘相关的性能问题研究
