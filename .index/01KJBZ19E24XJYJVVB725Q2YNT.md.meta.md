---
note: 01KJBZ19E24XJYJVVB725Q2YNT.md
title: 20230608 - 为sqle选择SQL引擎大模型
indexed_at: 2026-03-05T08:48:17.452147+00:00
---

## 摘要
笔记记录了为 sqle 项目选择 SQL 引擎大模型的调研过程，包括 HuggingFace 榜单候选模型（vicuna、falcon、mpt 等）、DIN-SQL 方案分析、SQL 数据集资源。包含 chatglm-6b 的初步测试结果，显示其对规则判断存在逻辑混乱问题，需要 fine-tuning。

## 关键概念
- DIN-SQL: Spider 榜单上的 NL2SQL 方案，通过在 prompt 中提供样例和思维链指导模型
- Text-to-SQL: 将自然语言转换为 SQL 查询的任务
- Fine-tuning: 对预训练模型进行微调以适应特定 SQL 任务
- Spider: NL2SQL 任务的权威评测榜单
- Vicuna: 基于 LLaMA 微调的对话模型，候选 SQL 引擎之一

## 关联笔记
- 01KJBZ1AAKZQX8R8K00CQV62YD.md: 后续对 chatglm 进行微调用于替换 SQL 审核引擎的实践
- 01KJBYY15XHA2SXQJ1QNJY255A.md: 早期测试 vicuna-13b+llama_index 的效果记录
- 01KJBZ22JR5ERJF0NH6TAAP190.md: SQL 测试数据集资源收集
