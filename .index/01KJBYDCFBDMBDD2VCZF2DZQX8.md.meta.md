---
note: 01KJBYDCFBDMBDD2VCZF2DZQX8.md
title: 20210606 - 从binlog中分离SQL, 进行统计
indexed_at: 2026-02-27T10:03:51.899261+00:00
---

用户要求我分析一篇关于 MySQL binlog 的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：20210606 - 从 binlog 中分离 SQL, 进行统计
2. 笔记内容：展示了如何使用 mysqlbinlog、awk 和 pt-query-digest 工具来解析和分析 MySQL binlog 文件

核心内容：
- 使用 mysqlbinlog 解析 binlog 文件
- 使用 awk 分离和合并 SQL 语句
- 使用 pt-query-digest 分析 SQL 分布

标签应该是：MySQL, binlog, SQL 分析, 性能诊断, 日志解析 等

关联笔记分析：
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 都与 MySQL 性能分析相关
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 都与 MySQL 相关
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: SQLLancer 学习 - 都与 SQL 分析相关

让我生成符合格式的元数据。
## 标签
MySQL, binlog, SQL 分析，性能诊断，日志解析，awk

## 摘要
介绍如何从 MySQL binlog 文件中提取和分离 SQL 语句。通过 mysqlbinlog、awk 和 pt-query-digest 工具链实现 SQL 的解析与分布统计。

## 关键概念
- mysqlbinlog: MySQL 官方提供的 binlog 解析工具
- Rows_query: binlog 中记录 SQL 语句的事件标记
- pt-query-digest: Percona Toolkit 的 SQL 分析工具，用于统计 SQL 分布

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同为 MySQL 性能诊断相关工具的使用
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同为 MySQL 底层机制研究与问题排查
