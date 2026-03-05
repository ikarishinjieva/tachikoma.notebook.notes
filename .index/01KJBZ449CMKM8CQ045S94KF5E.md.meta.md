---
note: 01KJBZ449CMKM8CQ045S94KF5E.md
title: 20231117 - 在Oracle并发压力中, 测试OMS性能 [1]
indexed_at: 2026-03-05T09:22:28.758051+00:00
---

## 摘要
记录在 Oracle 并发压力下使用 hammerDB 测试 OMS 性能的场景，通过队列状态监控定位到 TransactionScheduler 为阻塞点。使用 Arthas trace 工具逐层下挖，发现 submitTransaction 方法消耗 99.97% 耗时，进一步分析其数据计数限流机制。

## 关键概念
- TransactionScheduler: OMS 连接器中的事务调度器，负责梳理事务顺序、控制内存消耗和计算事务依赖
- RecordDispatcher: 记录分发器，负责将记录分发到不同队列（Ids/Records）
- 限流阀机制: TransactionScheduler_In_Rows/MaxRows，用于控制内存消耗，达到阈值时阻塞写入
- ConflictBroker: 冲突代理，计算事务依赖，当前序事务确认后事务才能输出
- Arthas trace: Java 诊断工具，用于追踪方法调用链路和统计各步骤耗时

## 关联笔记
- 01KJBZ3EQK7E6D34480VFBNXRD.md: 通过 Arthas 获取 OMS 队列监控的基础方法，与本笔记使用相同的技术手段
- 01KJBZ3RTRGDP0XM5JZAV07E42.md: 整理 OMS 增量复制链路，详细解释了 TransactionScheduler 和 RecordDispatcher 的作用
- 01KJBZ3VXDMGF2BT3VGE9QTJ4X.md: 使用 Arthas 导出 OMS 队列积压元素，与本笔记同属 OMS 性能调试场景
