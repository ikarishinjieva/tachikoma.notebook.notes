---
note: 01KJBZV3GZZ6MQD2PZXET9GAKM.md
title: 20250710 - 弘讯数据分析中发现的问题
indexed_at: 2026-03-05T11:45:59.714307+00:00
---

## 摘要
记录弘讯注塑机系统数据库分析中发现的 36 项数据问题，主要包括多表无数据、数据不一致、关联关系混乱等。这些问题影响生产过程的追溯和分析，需作为后续数据清洗的参考依据。

## 关键概念
- lotslog/workorder: 生产批次和工单表，存在起止时间不一致问题
- produce_craft: 生产工艺表，product_id 与上下游不一致
- opcua_data: OPCUA 数据点位配置，定义 PLC 地址映射
- operate_status: 机器操作状态表，有数据但状态值常量无说明

## 关联笔记
- 01KJBZV5A3J7TE29SPA3J0HRJ3.md: 引用本笔记作为数据问题汇总，继续按时间线整理生产过程数据
- 01KJBZV2RVXXM4F6GRNR83GD9G.md: 弘讯数据分析的初始笔记，包含 DDL 和数据结构说明
