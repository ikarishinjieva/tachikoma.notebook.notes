---
note: 01KJBZ9HS9G1R9WM8QRF6E3WRB.md
title: 20240604 - 阅读论文: LLM-R: Learning to Retrieve In-Context Examples for Large Language Models
indexed_at: 2026-03-05T09:59:12.049322+00:00
---

## 摘要
论文提出 LLM-R 方法，通过 LLM 对召回文档评分并将评分能力蒸馏给 retriever 模型，优化 in-context learning 中的文档选取。采用多轮迭代训练策略，每轮使用上一轮训练完成的模型，使训练数据动态变化以习得不同能力侧面。

## 关键概念
- in-context learning: 通过少量示例（few-shot）让模型在上下文中学习并完成新任务
- 知识蒸馏: 将 LLM 的文档评分能力转移到轻量级 retriever 模型
- 多轮迭代训练: 每轮使用上一轮训练完成的模型，训练数据动态变化
- reranking: 对召回文档进行重新排序和筛选的过程

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 检索系统对比笔记中明确引用 LLM-R 论文，记录其蒸馏 ranking 结果到 retriever 的思路
- 01KJBZ9CZVZM9SEVGF24T9XP2Z.md: 同样涉及使用蒸馏训练 reranking 模型，用 LLM 做排序使 KL 散度逼近
- 01KJBZ5PKDX5D2ZCWN2N2BK8TQ.md: 记录微调 retriever 方法，用大模型作为评审方使 retriever 输出接近大模型判断
