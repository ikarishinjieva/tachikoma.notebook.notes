---
note: 01KJBZ42D6BDXBV93D0EE08ZS2.md
title: 20231121 - 在Oracle并发压力中, 测试OMS性能 [2]
indexed_at: 2026-03-05T09:19:49.375764+00:00
---

## 摘要
场景 4（10 warehouse/10 vu）下数据同步慢，TransactionScheduler_OutQueue_Size 积压至 137090，但活跃数据库连接数仅维持在 20-40 之间。使用 Arthas trace 诊断 Writer.realWriteDML 方法，发现 Sink 处理过程中除数据库写入外还有其他逻辑导致连接利用率不高。

## 关键概念
- TransactionScheduler: 负责梳理事务顺序和控制内存限流的调度器
- sinkTask: 执行数据库 DML 写入的工作线程，默认 64 个
- realWriteDML: Writer 类中实际执行 DML 语句的核心方法
- 队列积压: TransactionScheduler 输出队列堆积，表明 Sink 消费速度跟不上生产速度

## 关联笔记
- 01KJBZ449CMKM8CQ045S94KF5E.md: 同一系列 [1]，记录 20231117 的场景 1-3 测试及 TransactionScheduler 阻塞点分析
- 01KJBZ428QFGM47RNAB68EERFS.md: 同一系列 [3]，记录 20231123 的后续性能测试
