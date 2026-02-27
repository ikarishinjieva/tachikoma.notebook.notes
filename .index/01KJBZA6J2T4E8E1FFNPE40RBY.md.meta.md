---
note: 01KJBZA6J2T4E8E1FFNPE40RBY.md
title: 20240618 - ChatDBA: 如何判断参考材料和问题的相关性 [2]
indexed_at: 2026-02-27T11:02:01.000575+00:00
---

用户要求我分析这篇关于 ChatDBA 和 MySQL 问题排查的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：研究 ChatDBA 系统中参考材料与问题的相关性判断
2. 背景：MySQL 执行 select 语句时 crash 的排查问题
3. 核心问题：答案中提到的"optimizer_switch"与 crash 的直接关系不大，需要间接推导
4. 过程：展示了一个生成空文档执行计划的 prompt 设计，用于让 AI 生成数据库问题排查计划

关键概念包括：
- ChatDBA（数据库 AI 助手）
- 相关性判断（参考材料与问题的关联度）
- 排查计划生成
- MySQL crash 诊断
- optimizer_switch

关联笔记：从列表中找与 MySQL、crash、诊断相关的笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了（可能与 crash 内存问题相关）

现在生成索引元数据。
## 标签
ChatDBA, MySQL 崩溃排查, 相关性判断, 排查计划生成, optimizer_switch, 工单分析

## 摘要
研究 ChatDBA 系统中参考材料与用户问题的相关性判断方法，以 MySQL select 语句 crash 排查为例。分析了 AI 生成的排查建议（如 optimizer_switch）与问题直接关联度较弱的情况，并展示了排查计划生成的 prompt 设计要求。

## 关键概念
- ChatDBA: 基于 AI 的数据库问题诊断助手，负责生成排查计划
- 相关性判断: 评估参考工单材料与当前问题的关联程度，避免间接推导
- 排查计划伪代码: 用 SWITCH/Case/THINKING/FIND/IF/RETURN 等函数表示诊断流程
- optimizer_switch: MySQL 优化器参数，与 crash 问题关联度较低需间接推导

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存问题排查与 MySQL crash 可能相关
- 01KJBYD9S0A3VZWG60QKDTAF49.md: InnoDB 刷脏页机制与数据库崩溃场景相关
