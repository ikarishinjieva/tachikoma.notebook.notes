---
note: 01KJBZD1P74N9Z643CH6RWT24X.md
title: 20240804 - 阅读论文*: REAPER: Reasoning based Retrieval Planning for Complex RAG Systems
indexed_at: 2026-03-05T10:28:52.824603+00:00
---

## 标签
RAG, 检索规划，REAPER, 数据增强，LLM, 推理

## 摘要
REAPER 是一个基于推理的检索规划器，使用小语言模型生成高效的检索计划（类似 CoT），决定从哪里检索什么关键字及调用哪个 Agent。通过 DQS、TEvo、TTG 三个模块生成多样化训练数据，解决模型偏差、工具描述依赖和任务泛化问题。

## 关键概念
- 检索计划：类似 CoT 的思考过程，决定从哪里检索什么关键字、调用哪个 Agent
- DQS (Diverse Query Sampler): 通过语义相似度分析筛选多样化查询样本，减少模型偏差
- TEvo (Tool Evolve): 对工具描述进行语义改写，增强模型对工具的理解泛化能力
- TTG (Tool-Task Generator): 将检索计划任务转换为多角度子任务，提升模型泛化能力

## 关联笔记
- 01KJBZD2RGYKMES9WAV22CF01C.md: ToolCoder 同样研究如何让模型使用 API 搜索工具，与 REAPER 的工具调用规划思路相关
- 01KJBZD3K42PBXXQ4KSTFQDP4N.md: CODEAGENT 的 Tool-Planning 策略与 REAPER 的检索计划生成机制相似
- 01KJBZG6ZPGE03511EZ5WVFPQP.md: 同样探讨工具辅助的 Agent 系统，涉及工具使用场景
