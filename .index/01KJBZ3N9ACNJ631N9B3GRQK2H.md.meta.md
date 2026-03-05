---
note: 01KJBZ3N9ACNJ631N9B3GRQK2H.md
title: 20231108 - OMS研究, 阻塞目标库, 对队列进行探索
indexed_at: 2026-03-05T09:13:15.735672+00:00
---

## 标签
OMS, 队列监控, 阻塞分析, store 进程, kafka connect, arthas

## 摘要
通过 arthas 观测 OMS 在 Oracle 阻塞时的队列状态，发现源端队列为 0、目标端队列正确阻塞。追踪目标端数据来源为 store 进程 (端口 17002)，发现子进程未正确处理继承端口的问题。分析源端 kafka connect 与 store 的连接关系及核心逻辑链路。

## 关键概念
- OMS 队列: 数据同步链路中各处理阶段间的缓冲队列，用于观测数据流转状态
- store 进程: OMS 目标端数据存储进程，负责向目标库写入数据
- kafka connect: 源端数据抽取框架，通过 LogMinerConnectorTask 从 store 获取数据
- arthas vmtool: Java 诊断工具，用于获取运行时对象状态和队列大小
- 阻塞模式: 目标库锁表时 OMS 队列的阻塞行为，用于验证同步链路的正确性

## 关联笔记
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 被引用，提供 arthas 队列监控脚本 get_status.sh 的使用方法
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: OMS 全量复制链路整理，涉及 store 进程和队列的详细说明
- 01KJBZ3P2X276MPVMQ02101059.md: OMS 增量复制链路日志分析，补充 OMS 日志系统的背景知识
