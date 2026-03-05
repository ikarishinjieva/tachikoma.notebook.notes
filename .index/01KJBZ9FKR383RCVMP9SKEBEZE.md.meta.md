---
note: 01KJBZ9FKR383RCVMP9SKEBEZE.md
title: 20240602 - 阅读论文*: UDAPDR: Unsupervised Domain Adaptation via LLM Prompting and Distillation of Rerankers
indexed_at: 2026-03-05T09:57:51.478746+00:00
---

## 摘要
UDAPDR 是一种无监督域适应方法，通过 LLM 生成查询数据训练多个 reranker，再将其知识蒸馏到单一 ColBERTv2 检索器。该方法利用多组提示词生成"问题 - 文档"训练数据，避免了目标领域标注数据的依赖，同时保持推理效率。

## 关键概念
- UDAPDR: 无监督域适应框架，通过 LLM 提示和 reranker 蒸馏训练检索器
- 两阶段 LLM 生成: 先用 GPT-3 生成初始查询，再用 Flan-T5 XXL 生成大量查询以降低成本
- Reranker 蒸馏: 将多个 reranker 的排序能力整合到单一检索器中，兼顾效果与效率
- 循环迭代优化: 阶段二到四可多次迭代，持续生成高质量查询并优化模型

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记，其中引用了 UDAPDR 作为 reranker 蒸馏到 retriever 的典型案例
- 01KJBZ9HS9G1R9WM8QRF6E3WRB.md: 同样采用 LLM 评分蒸馏给 retriever 的思路，但用于 in-context learning 场景
- 01KJBZ9CZVZM9SEVGF24T9XP2Z.md: 使用蒸馏训练 reranking 模型，让模型 KL 散度逼近 LLM 的排序能力
