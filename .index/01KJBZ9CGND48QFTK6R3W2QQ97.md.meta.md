---
note: 01KJBZ9CGND48QFTK6R3W2QQ97.md
title: 20240518 - UniMS-RAG: A Unified Multi-source Retrieval-Augmented Generation for Personalized Dialogue Systems
indexed_at: 2026-03-05T09:55:12.532020+00:00
---

## 标签
RAG, 个性化对话系统, UniMS-RAG, 评估标记, 联合训练, 自 refinement

## 摘要
UniMS-RAG 是一个统一多源检索增强生成框架，用于个性化对话系统，通过 Seq2Seq 方式统一知识源选择 (planner)、知识检索 (retriever) 和回复生成 (reader) 三个子任务。核心创新是引入行动标记和评估标记，配合自 refinement 机制确保生成回复与检索证据的一致性。

## 关键概念
- 行动标记: 用于决定下一步行动（选择哪个知识源）的特殊标记
- 评估标记: 表示对话上下文与检索证据相关性得分的监督信号
- 自 Refinement: 利用评估标记反馈重新评估生成回复与证据一致性的机制
- PerDS: 个性化知识增强对话回复生成任务 (Personalized Dialogue System)

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文笔记，在分类讨论中引用了 UniMS-RAG 作为应用案例
