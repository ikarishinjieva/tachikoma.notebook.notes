---
note: 01KJBZSKZ0GFCFP50QGNNNFB3Y.md
title: 20250506 - 阅读论文*: PROMPTCOT: Synthesizing Olympiad-level Problems for Mathematical Reasoning in Large Language Models
indexed_at: 2026-02-27T11:37:58.599861+00:00
---

用户要求我分析这篇关于 PROMPTCOT 论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 PROMPTCOT 论文的笔记
2. PROMPTCOT 是一种用于生成奥林匹克级别数学问题的方法
3. 核心步骤包括：概念提取、Rationale 生成、问题生成模型训练、拒绝采样等
4. 关键概念包括：rationale、拒绝采样、三元组（概念，rationale，问题）等

其他笔记列表主要是关于 MySQL、Clickhouse、系统调试等技术内容，与这篇 AI/LLM 论文笔记没有明显关联。

让我按照要求的格式输出：

## 标签
需要 3-6 个关键词
- 大语言模型
- 数据合成
- 数学推理
- PromptCot
- 拒绝采样
- 问题生成

## 摘要
2-3 句话概括核心内容

## 关键概念
3-5 个核心概念

## 关联笔记
从列表看，没有明显关联的笔记，因为其他笔记都是关于数据库、系统调试等内容## 标签
大语言模型，数据合成，数学推理，PromptCot，拒绝采样，问题生成

## 摘要
PROMPTCOT 是一种通过合成高质量数据来训练 LLM 生成奥林匹克级别数学问题的方法。核心流程包括从种子问题提取概念、生成 rationale、构造三元组数据、拒绝采样筛选，最终训练问题生成模型。

## 关键概念
- Rationale: 解释如何从给定概念出发一步步构造出问题的思维过程
- 三元组数据结构: 由概念、rationale、问题组成的训练数据单元
- 拒绝采样: 使用两个独立 LLM 评估器筛选高质量数据的机制
- 种子问题: 从 AoPS 等平台收集的已有奥林匹克级别问题作为数据生成基础

## 关联笔记
无
