---
note: 01KJBZ4J03FH868Z4ZDKGPQRDY.md
title: 20231226 - 在Oceanbase上, 开发MySQL的audit log [2]
indexed_at: 2026-02-27T10:41:30.896952+00:00
---

用户要求我分析这篇关于 OceanBase 开发 MySQL audit log 的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：在 OceanBase 数据库上开发 MySQL 的 audit log（审计日志）功能
2. **背景**：国测（国家测试/合规要求）
3. **需求**：审计操作和事件、审计规则和筛选、审计事件内容
4. **模块划分**：Audit Manager、Audit Module、Audit Logging Component、Configuration Component
5. **代码量要求**：总行数 18550 等
6. **步骤**：已完成的实验和后续计划

**标签**：应该包括 OceanBase、MySQL、审计日志、开发、国测等关键词

**摘要**：这篇笔记记录了在 OceanBase 数据库上开发 MySQL 审计日志功能的需求分析、模块设计和实现步骤。

**关键概念**：Audit Manager、Audit Module、审计事件、COM_Query 监听等

**关联笔记**：查看其他笔记列表，大部分是关于 MySQL、Clickhouse、性能分析等。这篇笔记是关于 OceanBase 开发 audit log 的，与其他笔记的关联性较弱。可能有关联的是：
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema的日常使用（都是关于数据库监控/审计相关）


- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 20210606 - 从 binlog 中分离 SQL, 进行统计（都涉及 SQL 监听和记录）

但这些关联都比较弱，因为这篇笔记主要是关于 OceanBase 的开发，而其他笔记大多是关于 MySQL、Clickhouse 的学习和问题排查。
## 标签
OceanBase, MySQL, 审计日志, 国测合规, 模块设计, 开发计划

## 摘要
记录在 OceanBase 数据库上开发 MySQL 审计日志功能的需求分析与模块设计。包括审计事件类型、规则配置、日志内容等需求，以及 Audit Manager、Audit Module 等四大模块划分和实现步骤。

## 关键概念
- Audit Manager: 负责管理和控制整个审计过程，配置审计规则
- Audit Module: 连接 ActionDB 监听数据库操作事件，执行实际审计
- Audit Logging Component: 将审计事件记录到数据库表并提供查询功能
- COM_Query: MySQL 协议中的查询命令，用于监听 SQL 操作
- 审计规则: 基于用户、IP 地址、数据库对象等因素定义审计条件

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用（同属数据库监控审计领域）
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 从 binlog 中分离 SQL 进行统计（涉及 SQL 监听与记录）
