---
note: 01KJBZ3GDKGKZ4M8GWVG6SPK2Z.md
title: 20231027 - 南航 ActionDB 3.x和4.x执行计划不一样
indexed_at: 2026-02-27T10:37:00.143504+00:00
---

用户要求我分析一篇关于南航 ActionDB 3.x 和 4.x 执行计划不一样的笔记，并生成索引元数据。

从笔记内容来看：
1. 标题提到"南航 ActionDB 3.x 和 4.x 执行计划不一样"
2. 内容是关于 `act_transaction` 表的定义，这是一个 OceanBase 数据库的表（从语法特征如 BLOCK_SIZE、COMPRESSION='zstd_1.3.8'、REPLICA_NUM 等可以看出）
3. 表按 Ins_Dt 字段进行范围分区

我需要根据这些信息生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从提供的列表中选择）

关于关联笔记，我需要看是否有与数据库、执行计划、OceanBase/ActionDB 相关的笔记。从列表中看：
- 大部分笔记是关于 MySQL、Clickhouse、Java 等技术
- 没有直接关于 OceanBase 或 ActionDB 的笔记
- 可能相关的有 MySQL 相关的笔记，但关联度不高

让我生成输出：
## 标签
OceanBase, 执行计划, 表分区, 南航, ActionDB, 索引

## 摘要
记录南航 ActionDB 系统中 act_transaction 表的表结构定义，该表采用按日期范围分区设计。用于分析 ActionDB 3.x 和 4.x 版本执行计划差异问题的基础资料。

## 关键概念
- act_transaction: 交易表，存储交易 ID、事件 ID、积分信息等核心业务数据
- 范围分区: 按 Ins_Dt 日期字段进行月度分区，便于历史数据管理
- 本地索引: index_Evt_ID 使用 BLOCK_SIZE 16384 的 LOCAL 索引设计

## 关联笔记
无
