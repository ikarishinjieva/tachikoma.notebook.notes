---
note: 01KJBYDAYBCJZXBGJFXXQ48F5X.md
title: 20210415 - Clickhouse MaterializedMySQL 学习
indexed_at: 2026-02-27T10:02:42.775037+00:00
---

用户希望我分析这篇关于 ClickHouse MaterializedMySQL 的笔记，并生成索引元数据。让我仔细阅读笔记内容，提取关键信息。

笔记主要内容：
1. ClickHouse MaterializedMySQL 引擎的文档学习
2. 代码入口和同步流程
3. 复制进度查看方法
4. Database UUID 查找方法
5. 故障恢复方法

核心概念：
- MaterializedMySQL（ClickHouse 的 MySQL 复制引擎）
- 虚拟列（_version, _sign）
- Binlog 复制
- GTID
- Buffer 机制

关联笔记：
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 20210501 - Clickhouse 集群 Distributed Engine 学习（都是 ClickHouse 相关）
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究（都是 ClickHouse 相关）
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+学习笔记（ClickHouse 学习笔记）

标签应该包括：ClickHouse, MaterializedMySQL, 数据复制, Binlog, MySQL, 同步

摘要：记录 ClickHouse MaterializedMySQL 引擎的工作原理，包括虚拟列机制、DDL/DML 转换规则、binlog 同步流程及代码入口分析。包含复制进度查看、元数据查询和故障恢复方法。

关键概念：
- MaterializedMySQL: ClickHouse 的 MySQL 实时同步引擎


- _version/_sign: 用于追踪事务版本和标记删除操作的虚拟列
- GTID: 全局事务标识符，用于精确对位复制位置
- Binlog Event: MySQL 二进制日志事件，驱动数据同步
- Buffer 机制: 批量处理 binlog 事件，优化写入性能

关联笔记：
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 同属 ClickHouse 引擎系列研究
- 01KJBYDBWTJGYAJWBZ93943E5R.md: ClickHouse 查询优化相关
- 01KJBYDD58AJTW5NKT3GET7B89.md: ClickHouse 学习笔记合集## 标签
ClickHouse, MaterializedMySQL, 数据复制, Binlog, MySQL 同步

## 摘要
记录 ClickHouse MaterializedMySQL 引擎的工作原理，包括虚拟列机制（_version/_sign）、DDL/DML 转换规则、binlog 同步流程及代码入口分析。包含复制进度查看、元数据查询和故障恢复方法。

## 关键概念
- MaterializedMySQL: ClickHouse 的 MySQL 实时同步引擎，通过 binlog 复制实现数据同步
- 虚拟列: _version 记录事务计数器，_sign 标记行删除状态（-1 表示删除）
- GTID: 全局事务标识符，用于 binlog 对位和复制进度追踪
- Buffer 机制: 按表缓存 binlog event，达到时间或大小阈值后批量 flush 到表
- DML 转换: MySQL DELETE/UPDATE 转换为带 _sign 的 INSERT 操作

## 关联笔记
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 同为 ClickHouse 引擎学习（Distributed Engine）
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 同为 ClickHouse 功能研究（Join）
- 01KJBYDD58AJTW5NKT3GET7B89.md: ClickHouse 学习笔记合集
