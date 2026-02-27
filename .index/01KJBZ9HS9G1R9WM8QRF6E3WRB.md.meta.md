---
note: 01KJBZ9HS9G1R9WM8QRF6E3WRB.md
title: 20240604 - 阅读论文: LLM-R: Learning to Retrieve In-Context Examples for Large Language Models
indexed_at: 2026-02-27T10:56:17.667040+00:00
---

用户希望我分析一篇关于 LLM-R 论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

笔记标题是关于 LLM-R 论文，主要内容是关于提升 in-context learning 时对召回文档的选取（reranking）。

主要思路：
1. 使用 LLM 对召回文档进行评分，将评分能力蒸馏给 retriever 模型
2. 使用多轮迭代，每一轮使用上一轮训练完成的模型

其他笔记列表主要是关于 MySQL、Clickhouse、数据库、系统性能等方面的内容，与这篇 LLM/检索增强生成的论文笔记没有明显关联。

让我生成索引：

## 标签
需要 3-6 个关键词：大语言模型、上下文学习、文档检索、知识蒸馏、reranking、迭代训练

## 摘要
2-3 句话概括核心内容

## 关键概念
3-5 个核心概念：in-context learning、知识蒸馏、retriever 模型、多轮迭代、文档评分

## 关联笔记
查看其他笔记列表，都是关于数据库、MySQL、Clickhouse、系统性能等内容，与这篇 LLM 论文笔记没有明显关联，所以应该是「无」## 标签
大语言模型，上下文学习，文档检索，知识蒸馏，Reranking，迭代训练

## 摘要
该论文提出 LLM-R 方法，通过 LLM 对召回文档评分并将能力蒸馏给 retriever 模型，优化 in-context learning 中的 few-shot 示例选择。采用多轮迭代训练策略，每轮使用上一轮模型，利用训练数据变化获取不同维度的能力。

## 关键概念
- In-context Learning: 通过提示词中的 few-shot 示例让模型学习完成任务
- 知识蒸馏: 将 LLM 的文档评分能力迁移到轻量级 retriever 模型
- Reranking: 对召回文档进行重新排序和筛选，选取最优示例
- 多轮迭代训练: 每轮使用上一轮模型，训练数据动态变化以获取多维度能力

## 关联笔记
无
