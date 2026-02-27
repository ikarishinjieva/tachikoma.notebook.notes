---
note: 01KJBYDACWHS9VPYASX7JF6Q61.md
title: 20210414 - 华夏银行迁移工具评估
indexed_at: 2026-02-27T10:01:52.380660+00:00
---

用户要求我分析这篇关于华夏银行迁移工具评估的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于数据库迁移工具评估的笔记
2. 主要涉及 Oracle 到 MySQL 的迁移
3. 提到了"龙跃"产品白皮书
4. 内容包括：迁移评估、智能转换、数据同步、数据校验、安全回退等模块

现在我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

看其他笔记列表，寻找关联：
- 有几篇关于 Clickhouse 的笔记
- 有关于 MySQL 刷盘、刷脏页的笔记
- 有关于 performance_schema 的笔记
- 有关于 SQLLancer 学习的笔记
- 有关于 DDL 过程梳理的笔记

这篇笔记主要是关于数据库迁移工具（龙跃）的评估，涉及 Oracle 到 MySQL 的迁移。相关的可能有：
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习（时间接近，但主题不同）
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习（也是数据库工具相关）


- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（银行项目相关）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（MySQL 相关）
- 01KJBYDKM74PTS0GRF808YHQR7.md: 20210628 - DDL 过程梳理（数据库相关）

最相关的应该是银行项目相关的笔记和数据库工具学习的笔记。

时间上 20210414 接近的有 20210415 的 Clickhouse MaterializedMySQL 学习，内容上涉及数据库迁移和 Oracle 到 MySQL 的转换，这些都可以作为参考。
## 标签
数据库迁移，Oracle 到 MySQL, 龙跃工具，迁移评估，数据同步，安全回退

## 摘要
记录华夏银行数据库迁移项目中对龙跃迁移工具的评估内容，涵盖迁移评估、智能转换、数据同步、数据校验及安全回退等核心功能。该工具支持 Oracle 到 MySQL 的异构数据库迁移，包括表结构自动转换、SQL 语法转换、存储过程转 Java 代码等能力。

## 关键概念
- 迁移评估: 对数据库版本、对象兼容性、应用 SQL 改造成本及工作量进行综合评估
- 智能转换: 自动将 Oracle 的表结构、SQL 语法、PLSQL 存储过程转换为 MySQL 兼容格式
- 数据同步: 支持全量同步和增量同步两种方式，根据停机窗口和数据量选择
- 数据校验: 通过统计对比和逐行算法校验确保源端和目标端数据一致性
- 安全回退: 支持异构数据库双向同步，实现异常场景下的快速切换和回退

## 关联笔记
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 同为 2021 年 4 月中旬的数据库迁移相关技术学习（MaterializedMySQL）
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 同为数据库工具类产品学习（SQLLancer）
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同为银行客户（农行）的 MySQL 相关问题记录
