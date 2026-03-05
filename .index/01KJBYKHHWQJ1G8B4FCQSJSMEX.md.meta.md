---
note: 01KJBYKHHWQJ1G8B4FCQSJSMEX.md
title: 20220811 - golang产生对MySQL的并发连接代码
indexed_at: 2026-03-05T08:06:16.484262+00:00
---

## 标签
Golang, MySQL, 并发连接, 压力测试, goroutine, 数据库驱动

## 摘要
使用 Go 1.13 编写并发测试程序，通过 goroutine 循环创建 10000 个 MySQL 连接进行压力测试。代码包含连接池并发查询和版本检测，附带完整源码包。

## 关键概念
- goroutine: Go 语言轻量级线程，用于并发执行数据库连接
- 并发连接测试: 同时建立大量数据库连接以测试服务器承载能力
- go-sql-driver/mysql: Go 语言 MySQL 数据库驱动库
- 压力测试: 通过高并发连接验证数据库性能和稳定性

## 关联笔记
- 01KJBYKK56GGEG65TSY8WHHZXJ.md: 引用本代码向 MySQL slave 建立 40000 根空闲连接进行压力测试
