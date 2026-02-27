---
note: 01KJBZ42D6BDXBV93D0EE08ZS2.md
title: 20231121 - 在Oracle并发压力中, 测试OMS性能 [2]
indexed_at: 2026-02-27T10:39:36.893200+00:00
---

用户要求我分析这篇关于 Oracle 并发压力测试 OMS 性能的笔记，生成索引元数据。让我仔细分析笔记内容：

1. **主题**：Oracle 并发压力测试中 OMS（Oceanbase Migration Service?）性能问题排查
2. **场景**：10 warehouse, 10 vu, buildschema 数据初始化，数据同步慢
3. **现象**：
   - 队列积压（TransactionScheduler_OutQueue_Size=137090）
   - 活跃连接数少（维持在 20-40 之间）
   - 数据库连接不满
   - sink 处理停留在 realWriteDml

4. **工具使用**：
   - Arthas（vmtool, trace）
   - OBclient show full processlist

5. **核心问题**：TransactionScheduler 队列积压但数据库连接使用率不高，怀疑 Sink 处理中还有其他逻辑

现在生成元数据：

**标签**：OMS, 性能测试，Oracle, Arthas, 队列积压，JDBC

**摘要**：在 Oracle 并发压力测试中发现 OMS 数据同步慢的问题。通过 Arthas 诊断发现 TransactionScheduler 队列积压但数据库连接使用率不高，sink 处理停留在 realWriteDml 方法。

**关键概念**：
- TransactionScheduler: OMS 中负责事务调度的组件，其输出队列出现积压
- Arthas vmtool: 用于获取 JVM 中对象实例信息，诊断连接锁定状态


- RecordDispatcher/ETLProcessor/StoreSource: OMS 数据同步链路中的关键处理组件

**关联笔记**：这些笔记涉及 MySQL 刷盘、连接池、binlog 分析等数据库性能相关内容，但与当前 OMS 性能问题的直接关联不明显。
## 标签
OMS, 性能测试, Oracle, Arthas, 队列积压, JDBC

## 摘要
在 Oracle 并发压力测试中发现 OMS 数据同步慢的问题。通过 Arthas 诊断发现 TransactionScheduler 输出队列积压但数据库连接使用率不高，sink 处理停留在 realWriteDml 方法，怀疑 Sink 处理中存在其他逻辑导致连接利用率低。

## 关键概念
- TransactionScheduler: OMS 中负责事务调度的组件，其输出队列出现积压（137090）
- Arthas vmtool: 用于获取 JVM 中对象实例信息，诊断连接锁定状态
- RecordDispatcher/ETLProcessor: OMS 数据同步链路中的核心处理组件

## 关联笔记
无
