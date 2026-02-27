---
note: 01KJBZRPASEJVBQPE28AAP50YZ.md
title: 20250227 - 阅读论文*: 迈向复现 OpenAI o1 的一小步：Steiner 开源模型解析
indexed_at: 2026-02-27T11:34:11.320343+00:00
---

用户要求我分析一篇关于 OpenAI o1 模型复现的论文解析笔记，并生成索引元数据。让我仔细阅读笔记内容，然后按照要求的格式输出。

笔记核心内容：
1. 关于 o1 模型的 reasoning tokens 线性增长分析
2. 如何构造带 backtracking 的 CoT 流程
3. RL 阶段的 reward 设计

标签应该包括：o1 模型、推理、CoT、强化学习、backtracking 等

摘要需要概括这篇笔记是关于 Steiner 开源模型对 o1 的复现分析，包括推理机制和数据合成方法。

关键概念：
- reasoning tokens
- backtracking
- CoT (Chain of Thought)
- DAG (有向无环图)
- Step-Level Reward

关联笔记：查看其他笔记列表，都是关于 MySQL、Clickhouse、数据库、系统性能等方面的笔记，与这篇 AI/LLM 推理的笔记没有明显关联。所以应该写「无」。

让我按照格式输出。## 标签
o1 模型，推理机制，Chain of Thought，强化学习，backtracking，数据合成

## 摘要
本文解析了 Steiner 开源模型复现 OpenAI o1 的技术方案，分析了 o1 的线性推理遍历机制及其优势。介绍了构造带 backtracking 的 CoT 数据合成方法和基于 DAG 的 step-level reward 设计。

## 关键概念
- reasoning tokens: 用于推断 o1 决策树遍历方式的线性增长指标
- backtracking: 允许回溯的推理路径，比纯树形搜索更自由
- DAG: 将 reasoning steps 聚类后构建的有向无环图，用于采样推理路径
- Step-Level Reward: 基于节点入边/出边数、距离问题/答案的距离加权计算 reward

## 关联笔记
无
