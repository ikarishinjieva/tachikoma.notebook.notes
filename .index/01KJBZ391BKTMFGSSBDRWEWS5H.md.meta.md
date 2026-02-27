---
note: 01KJBZ391BKTMFGSSBDRWEWS5H.md
title: 20230830 - 随机SQL生成
indexed_at: 2026-02-27T10:34:41.396680+00:00
---

用户要求我分析一篇关于随机 SQL 生成的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题是关于随机 SQL 生成工具（randgen）
2. 涉及 MariaDB 的随机查询生成器
3. 包含命令行示例、原理说明、规则调整方法
4. 用于 SQL audit 训练

我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

查看其他笔记列表，寻找关联：
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习（SQL 相关工具，可能有关联）
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 20210606 - 从 binlog 中分离 SQL, 进行统计（SQL 分析相关）

其他笔记大多是关于 Clickhouse、内存、刷盘等主题，与随机 SQL 生成关联不大。

让我生成输出：## 标签
随机 SQL 生成, MariaDB, randgen, SQL 测试, 语法文件, 审计训练

## 摘要
介绍使用 MariaDB randgen 工具通过.yy 语法文件随机生成 SQL 的方法。包含命令行参数示例及如何通过调整语法规则生成恒等条件，便于 SQL audit 训练时识别特定类型的 SQL。

## 关键概念
- randgen: MariaDB 的随机查询生成工具，用于生成测试 SQL
- .yy 语法文件: 定义 SQL 生成规则的语法配置文件
- DEGENERATE 标签: 用于标记恒等条件，便于审计训练时识别结果
- invariant: 语法文件中的标记，表示生成不变量条件

## 关联笔记
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 同为 SQL 测试/分析工具的学习笔记
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 都涉及 SQL 分析与统计相关技术
