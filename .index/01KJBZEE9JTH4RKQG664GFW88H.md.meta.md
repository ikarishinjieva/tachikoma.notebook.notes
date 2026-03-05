---
note: 01KJBZEE9JTH4RKQG664GFW88H.md
title: 20240816 - mysql crash 海南银行
indexed_at: 2026-03-05T10:34:59.041259+00:00
---

## 摘要
海南银行 MySQL 8.0.26 实例崩溃的 core dump 分析，通过分离线程堆栈定位锁等待链。核心问题是 LOCK_status → srv_innodb_monitor_mutex → trx_sys 的等待链，多个 THD::release_resources 线程被阻塞。

## 关键概念
- LOCK_status: MySQL 服务器状态全局锁，多个线程等待此锁释放资源
- srv_innodb_monitor_mutex: InnoDB 监控器互斥锁，用于导出 InnoDB 状态信息
- trx_sys: InnoDB 事务系统锁，控制事务分配和 MVCC 版本可见性
- THD::release_resources: MySQL 线程资源释放函数，关闭连接时需获取多个锁
- 堆栈分离脚本: 将 core dump 中每个线程的堆栈分离成独立文件便于排查

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同样是 MySQL crash coredump 分析笔记，涉及 InnoDB 崩溃堆栈排查
