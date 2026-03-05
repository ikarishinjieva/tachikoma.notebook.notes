---
note: 01KJBZ6V4HFTNCBMEZSA71YAKB.md
title: 20240228 - 对embedding进行微调 - 增加训练数据
indexed_at: 2026-03-05T09:38:15.791485+00:00
---

## 标签
MySQL, 故障排查，主从复制，锁等待，OOM, 复制延迟

## 摘要
收集了 100+ 个 MySQL 数据库常见故障场景，涵盖主从复制异常、锁等待超时、OOM 崩溃、连接错误、复制延迟等典型问题。作为 ChatDBA 微调训练数据，用于增强模型对数据库故障诊断的回答能力。

## 关键概念
- 主从复制: MySQL 通过 binlog 和 relay log 实现主库与从库之间的数据同步
- Slave_IO_Running: 从库 IO 线程状态，负责从主库读取 binlog 并写入 relay log
- 锁等待超时: 事务等待锁释放超过 innodb_lock_wait_timeout 阈值时报错
- 半同步复制: 主库写入后需等待至少一个从库确认才返回成功的复制模式
- GTID: 全局事务标识符，用于追踪和保证主从复制的一致性

## 关联笔记
- 01KJBZA7W62X0HJ2HGDXSYNQ7Z.md: 包含主从延迟 60 多个 binlog 的排查案例和 ChatDBA 测试流程
- 01KJBZED41K5CMAYSC5QKC2E76.md: 分析 MySQL mutex 锁的 signal_count 机制，涉及锁等待底层原理
- 01KJBZAD3XNGKCT09G5YRWR7WP.md: 包含 MySQL crash 排查方法，与本笔记中崩溃类问题相关
