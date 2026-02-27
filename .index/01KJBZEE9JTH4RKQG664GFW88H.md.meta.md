---
note: 01KJBZEE9JTH4RKQG664GFW88H.md
title: 20240816 - mysql crash 海南银行
indexed_at: 2026-02-27T11:12:34.842320+00:00
---

用户要求我分析一篇关于 MySQL 崩溃的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是关于海南银行 MySQL 崩溃的排查笔记
2. 涉及 GDB 堆栈分析
3. 主要问题是 LOCK_status 相关的线程等待问题
4. 涉及 InnoDB 的 srv_innodb_monitor_mutex 锁等待
5. 有具体的线程堆栈信息

现在我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：从列表中找 0-3 篇相关的

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页 - 有一定关联，都是 InnoDB 相关
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 日常使用 - 有关联，涉及性能排查
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 有一定关联
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 有一定关联

最相关的是 performance_schema 那篇，因为笔记中涉及性能诊断。

让我生成输出：## 标签
MySQL 崩溃，锁等待，InnoDB，堆栈分析，LOCK_status，srv_innodb_monitor_mutex

## 摘要
海南银行 MySQL 崩溃排查，发现多个线程等待 LOCK_status，其中一个线程持有 LOCK_status 但等待 srv_innodb_monitor_mutex。通过 GDB 堆栈分析定位到 PFS_status_variable_cache 和 InnoDB 状态导出相关的锁竞争问题。

## 关键概念
- LOCK_status: MySQL 服务器状态相关的互斥锁，多个线程竞争会导致阻塞
- srv_innodb_monitor_mutex: InnoDB 监控相关的互斥锁，导出 InnoDB 状态时需要获取
- PFS_status_variable_cache: Performance Schema 中用于缓存状态变量的组件
- THD::release_resources: MySQL 线程释放资源时可能等待 LOCK_status

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 都涉及 performance_schema 相关的性能排查
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 都涉及 InnoDB 内部机制研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 都是 MySQL 刷脏页/锁相关的深度排查笔记
