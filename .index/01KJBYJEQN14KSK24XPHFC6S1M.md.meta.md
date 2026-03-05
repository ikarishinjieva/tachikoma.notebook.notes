---
note: 01KJBYJEQN14KSK24XPHFC6S1M.md
title: 20220528 - page_cleaner日志的模拟方法
indexed_at: 2026-03-05T07:59:27.602050+00:00
---

## 摘要
研究 InnoDB page_cleaner 日志连续打印周期的模拟方法。通过 GDB 调试在 buf0flu.cc 关键位置注入延迟，复现日志打印条件并分析内部周期参数。

## 关键概念
- page_cleaner: InnoDB 负责协调缓冲池脏页刷盘的后台线程
- buf0flu.cc: MySQL 5.7 中缓冲池刷盘逻辑的源代码文件
- warn_interval: page_cleaner 警告计数器的间隔阈值
- curr_time - next_loop_time: 实际循环时间与预期时间的差值

## 关联笔记
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同属 MySQL 刷盘机制研究，包含 buf_flush_page_cleaner 线程堆栈分析
