---
note: 01KJBYJDY946YJAPPA1981SMS8.md
title: 20220508 - 杭州银行 MySQL slave 崩溃
indexed_at: 2026-02-27T10:16:33.258660+00:00
---

用户要求我分析一篇关于杭州银行 MySQL slave 崩溃的笔记，并生成索引元数据。我需要按照指定的 Markdown 格式输出标签、摘要、关键概念和关联笔记。

让我分析笔记内容：

1. 主要内容：杭州银行 MySQL slave 崩溃分析，涉及 InnoDB 信号量等待问题
2. 核心问题：LOCK_SYS_WAIT 信号量等待超过 900 秒导致服务器崩溃
3. 技术分析：涉及锁等待、semaphore wait、lock_wait_timeout_thread 等
4. 关联笔记：需要查看提供的笔记列表中哪些与 MySQL、InnoDB、锁相关

相关关联笔记：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理 (InnoDB 相关)
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 (MySQL 相关)
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 (MySQL 相关)
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 (可能与调试相关)

现在生成元数据。## 标签
MySQL, InnoDB, 信号量等待, 锁等待, 崩溃分析, slave

## 摘要
分析杭州银行 MySQL slave 因 InnoDB 信号量等待超时（LOCK_SYS_WAIT 超过 900 秒）导致崩溃的问题。通过日志分析定位到 lock_wait_timeout_thread 可能卡顿，但无法确定锁持有方，推论锁状态可自行恢复。

## 关键概念
- LOCK_SYS_WAIT: InnoDB 锁系统等待互斥锁，超时 900 秒触发崩溃
- lock_wait_timeout_thread: 负责锁等待超时的后台线程，可能卡顿但无有效监控指标
- INNODB MONITOR OUTPUT: InnoDB 诊断输出，日志中出现 207 轮等待记录
- TRX_SYS / Global_exclusive_latch_guard: 日志中涉及的其他两类锁

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同属 InnoDB 刷脏页与前台处理机制研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同属 MySQL 刷脏页与性能问题研究
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同属 MySQL 刷盘慢导致 SQL 卡顿类问题
