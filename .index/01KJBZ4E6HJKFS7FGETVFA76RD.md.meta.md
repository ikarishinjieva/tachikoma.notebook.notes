---
note: 01KJBZ4E6HJKFS7FGETVFA76RD.md
title: 20231205 - 整理OMS的全量验证链路
indexed_at: 2026-03-05T09:23:43.600059+00:00
---

## 标签
OMS, 数据验证，全量校验，Oracle, 数据同步，源码分析

## 摘要
记录 OMS 全量验证组件的运行配置与核心架构，包括 Java 进程参数、checker.conf 配置项及数据源连接信息。梳理 Split 阶段的 SourceGroup、SourceExchangeInfo、CacheCollection 等核心概念与线程模型。

## 关键概念
- SourceGroup: 包含源端和目标端多个数据源的逻辑分组
- InfoExchanger/SliceExchanger: 处理源端和目标端之间的差异协调（如表结构不同导致的查询条件差异）
- CacheCollection: 用于 Split 和 Verify 过程中的临时数据存储
- ConditionSourceConnectTask: 针对单个数据源执行数据读取的线程
- TriggerSliceProvider/PassiveSliceProvider: 主动/被动切片划分器，分别用于源端和目标端

## 关联笔记
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: 前一天 (12 月 4 日) 的姊妹篇，记录 OMS 全量复制链路的处理器与切分逻辑
- 01KJBYWKGYZ1ZAP7STRSZWAVZJ.md: 更早的 OMS checker 代码分析笔记，包含启动脚本与 jar 包依赖详情
