---
note: 01KJBZETD71Z66AHRQNQ181WNG.md
title: 20240826 - 阅读论文: AUTOMATIC CHAIN OF THOUGHT PROMPTING IN LARGE LANGUAGE MODELS
indexed_at: 2026-03-05T10:38:27.930507+00:00
---

## 摘要
文档提出 Auto-CoT 方法，通过问题聚类 (Sentence-BERT + k-means) 和演示采样两阶段自动构建 CoT 样例。核心缺陷是聚类空间与推理深度无必然关联，语义相似不代表推理深度类似。

## 关键概念
- Auto-CoT: 自动构建思维链演示的方法，无需人工标注
- 问题聚类: 使用 Sentence-BERT 编码 + k-means 将问题分组确保多样性
- Zero-Shot-CoT: 通过"Let's think step by step"让 LLM 自动生成推理链
- 演示采样: 从每个聚类中选择代表性问题生成 CoT 样例

## 关联笔记
- 01KJBZQZWKZXG432876BXTV83W.md: ZEUS 方法同样使用聚类选择 CoT 样例，但基于不确定性筛选
- 01KJBZ9R4MN3T2EN6DV5JDPR7F.md: Active Prompt 讨论如何选择最有用的问题作为 CoT 的 few-shot 样例
