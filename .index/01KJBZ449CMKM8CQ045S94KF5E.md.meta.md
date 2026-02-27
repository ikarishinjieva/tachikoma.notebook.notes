---
note: 01KJBZ449CMKM8CQ045S94KF5E.md
title: 20231117 - 在Oracle并发压力中, 测试OMS性能 [1]
indexed_at: 2026-02-27T10:39:51.661638+00:00
---

用户希望我分析这篇关于 Oracle 并发压力下测试 OMS 性能的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：在 Oracle 并发压力测试中，使用 hammerDB 和 TPCC 对 OMS（OceanBase Migration Service）进行性能测试和瓶颈分析

2. **核心内容**：
   - 使用 hammerDB 进行 TPCC 压力测试
   - 发现同步慢，通过队列状态判断阻塞点在 TransactionScheduler
   - 使用 arthas trace 功能定位性能问题
   - 问题集中在 submitTransaction 过程

3. **关键概念**：
   - hammerDB：数据库性能测试工具
   - TPCC：事务处理性能测试基准
   - OMS：OceanBase Migration Service
   - arthas：Java 诊断工具
   - TransactionScheduler：事务调度器

4. **关联笔记分析**：
   - 这篇笔记是关于 2023 年 11 月的 OMS 性能测试
   - 其他笔记大多是 2021 年的 MySQL、Clickhouse 相关内容
   - 没有直接关联的笔记（时间差距大，主题也不同 - 这篇是 OceanBase OMS，其他多是 MySQL/Clickhouse）

让我生成索引：

## 标签
OMS, 性能测试, Oracle, hammerDB, arthas, 瓶颈分析

## 摘要


在 Oracle 并发压力环境下对 OMS 进行性能测试，通过队列状态监控发现 TransactionScheduler 成为阻塞点。利用 arthas trace 工具深入追踪，定位到 submitTransaction 方法存在性能瓶颈。

## 关键概念
- hammerDB: 数据库压力测试工具，用于生成 TPCC 负载
- arthas: Java 应用诊断工具，用于方法调用追踪和性能分析
- TransactionScheduler: 负责事务调度和提交的核心组件
- TPCC: 行业标准的事务处理性能测试基准
- OMS: OceanBase 数据迁移服务

## 关联笔记
无（现有笔记主要涉及 MySQL 和 Clickhouse，与本次 OMS 性能分析无直接关联）
## 标签
OMS, 性能测试, Oracle, hammerDB, arthas, 瓶颈分析

## 摘要
在 Oracle TPCC 并发压力测试下分析 OMS 同步延迟问题，通过队列状态判断阻塞点位于 TransactionScheduler。使用 arthas trace 定位到 submitTransaction 方法耗时过长，需进一步下挖分析。

## 关键概念
- hammerDB: 数据库压力测试工具，用于生成 TPCC 负载
- arthas trace: Java 应用诊断工具，可追踪方法调用链路和耗时
- TransactionScheduler: 事务调度器，负责事务提交调度
- TPCC: 事务处理性能委员会基准测试，模拟 OLTP 场景
- OMS: OceanBase 迁移服务，用于数据同步和迁移

## 关联笔记
无
