---
note: 01KJBZ3FQT8FKX47BF6Q8C4X7J.md
title: 20231027 - 对OMS的日志系统分析
indexed_at: 2026-03-05T09:07:09.837203+00:00
---

## 标签
OMS, 日志分析, Kafka Connect, logback, traceId, arthas

## 摘要
分析 OMS 系统的日志架构，包括 Kafka Connect、CM(jetty)、Ghana、store 等组件的日志配置和文件结构。通过 arthas 和 traceId 追踪跨组件调用链路，提出日志改进建议。

## 关键概念
- arthas: Java 诊断工具，用于查看日志输出堆栈和线程信息
- traceId: 跨服务调用的追踪标识，用于关联不同组件的日志
- logback: Java 日志框架，配置日志输出格式和文件轮转策略
- ConnectDRCDeliver: Kafka Connect 进程，负责从 Oracle 订阅增量数据
- 日志轮转: 按时间或大小滚动日志文件，控制磁盘占用

## 关联笔记
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 同系列笔记，使用 arthas 监控 OMS 队列状态
- 01KJBZ3KF8JB5YRT07J6JXC86C.md: 同系列笔记，通过 traceId 诊断 OMS 界面流程
- 01KJBZ3RTRGDP0XM5JZAV07E42.md: 同系列笔记，整理 OMS 增量复制链路
