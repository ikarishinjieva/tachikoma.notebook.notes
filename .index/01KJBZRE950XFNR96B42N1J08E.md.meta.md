---
note: 01KJBZRE950XFNR96B42N1J08E.md
title: 20250211 - 阅读论文*: OPEN-RAG: Enhanced Retrieval-Augmented Reasoning with Open-Source Large Language Models
indexed_at: 2026-03-05T11:27:42.992306+00:00
---

## 摘要
OPEN-RAG 结合检索增强生成（RAG）与专家混合（MoE）架构，通过反射标记控制检索行为和评估答案质量。采用共享 FFN 参数配合独立适配器的专家设计，以及动态路由机制选择激活专家，实现参数高效和计算高效的复杂推理。

## 关键概念
- 反射标记：控制检索触发、评估检索结果、排序答案和评估模型不确定性的特殊标记
- MoE 架构：用专家混合层替代标准 FFN 层，所有专家共享冻结参数并拥有独立可训练适配器
- 动态路由：可学习的路由模块根据注意力输出选择激活 top-k 个专家
- 检索增强生成 (RAG)：从外部知识库检索文档片段辅助模型生成答案

## 关联笔记
- 01KJBZR8RYBBRTNE7SXJ3W51GG.md: Lamini-1 同样使用 MoE 架构和动态路由机制进行专家选择
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 是另一种 RAG 优化方案，通过草稿机制降低计算成本
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 方案同样对 RAG 流程进行精细化设计和优化
