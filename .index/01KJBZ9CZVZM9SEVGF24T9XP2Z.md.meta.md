---
note: 01KJBZ9CZVZM9SEVGF24T9XP2Z.md
title: 20240520 - 阅读论文: XRICL: Cross-lingual Retrieval-Augmented In-Context Learning for Cross-lingual Text-to-SQL Semantic Par
indexed_at: 2026-03-05T09:55:54.637030+00:00
---

## 摘要
XRICL 提出跨语言 Text-to-SQL 解决方案，采用类似 RAG 的流程架构。核心创新包括使用蒸馏训练使 reranking 模型逼近 LLM 排序能力，以及将翻译任务与生成任务融合到 few-shot 提示词中实现协同优化。

## 关键概念
- 跨语言 Text-to-SQL: 将自然语言问题转换为 SQL 查询，支持源语言与目标语言不同的场景
- 蒸馏训练: 用大语言模型作为教师，让 reranking 模型的 KL 散度逼近 LLM 的排序分布
- Few-shot 提示词融合: 在同一提示词中同时包含翻译样例和 SQL 生成样例，实现两任务协同

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记，在应用部分引用了 XRICL 作为 RAG 在 Text-to-SQL 领域的案例
- 01KJBZRVT5F0HWQY45ZK497JMA.md: SQL-o1 论文笔记，同属 Text-to-SQL 领域，使用 MCTS 搜索方法生成 SQL
- 01KJBZPTT8A443T3Q5V8SWX4QJ.md: KeyInst 论文笔记，同属 Text-to-SQL 领域，通过关键字指令改善 SQL 生成
