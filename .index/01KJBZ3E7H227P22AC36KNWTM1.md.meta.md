---
note: 01KJBZ3E7H227P22AC36KNWTM1.md
title: 20231024 - arthas 学习
indexed_at: 2026-03-05T09:05:28.373339+00:00
---

## 摘要
Arthas 是阿里巴巴开源的 Java 诊断工具，提供 thread、vmtool、trace、watch 等命令实现生产环境实时监控。支持线程分析、对象实例获取、方法调用追踪、动态类替换等功能，用于问题定位和性能优化。

## 关键概念
- thread: 分析线程状态，定位阻塞线程和最繁忙线程
- vmtool: 通过 OQL 表达式获取 JVM 中对象实例，支持 forceGc
- trace: 追踪方法调用路径并统计各节点耗时
- watch: 观察方法的参数、返回值、异常等上下文信息
- retransform: 动态替换类文件实现热更新

## 关联笔记
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 同系列笔记，展示通过 arthas vmtool 获取 OMS 队列监控的实际案例
- 01KJBZ449CMKM8CQ045S94KF5E.md: 使用 arthas trace 和 watch 命令分析消费延迟和队列状态
- 01KJBZ4E6HJKFS7FGETVFA76RD.md: 详细记录 arthas thread 命令的输出分析和线程问题定位
