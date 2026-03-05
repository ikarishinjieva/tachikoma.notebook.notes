---
note: 01KJBZPTT8A443T3Q5V8SWX4QJ.md
title: 20250103 - 阅读论文: KeyInst: Keyword Instruction for Improving SQL Formulation in Text-to-SQL
indexed_at: 2026-03-05T11:10:36.911117+00:00
---

## 标签
Text-to-SQL, KeyInst, 关键词指令, SQL 生成, 错误修复, Prompt 工程

## 摘要
KeyInst 通过要求 LLM 在生成 SQL 前先思考并给出合适的 SQL 关键字（如 JOIN、ORDER BY 等），来减少关键字逻辑缺失或误用的错误。该方法主要针对高优先级 SQL 关键词导致的查询结果错误，但无法解决 Schema Linking、值匹配等其他类型错误。

## 关键概念
- KeyInst: 一种通过显式提示 LLM 关注关键 SQL 关键词来改进 SQL 生成的方法
- Schema Linking 错误: LLM 未能正确将自然语言实体映射到数据库表和列的错误
- SQL 关键字逻辑错误: 关键 SQL 关键字（GROUP BY、WHERE 等）缺失或误用导致的查询错误
- 值匹配错误: LLM 无法正确识别 NLQ 中的值或转换为合适 SQL 格式的错误

## 关联笔记
- 01KJBZRVT5F0HWQY45ZK497JMA.md: SQL-o1 同样针对 Text-to-SQL 任务，使用 MCTS 搜索和自奖励机制优化 SQL 生成
- 01KJBZSH9AJT9PM0QF3R1R4Y35.md: SQL-R1 使用强化学习训练 Text2SQL 模型，与 KeyInst 同为改进 SQL 生成的不同方法
