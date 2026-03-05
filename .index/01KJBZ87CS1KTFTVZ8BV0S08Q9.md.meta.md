---
note: 01KJBZ87CS1KTFTVZ8BV0S08Q9.md
title: 20240504 - 阅读论文: QA-RAG
indexed_at: 2026-03-05T09:48:24.494035+00:00
---

## 标签
RAG, 文档召回, HyDE, LLM 微调, 检索增强

## 摘要
QA-RAG 使用微调后的 LLM 生成 fake answer，将 fake answer 与原始 query 结合进行文档召回。该方法类似于 HyDE，但 HyDE 使用通用 LLM 会引入杂音知识，而 QA-RAG 通过微调减少这一问题。

## 关键概念
- Fake Answer: 用微调 LLM 生成的假设性答案，用于增强文档召回
- 文档召回: 将 query 与 fake answer 结合，在向量数据库中检索相关文档
- HyDE: Hypothetical Document Embeddings，使用通用 LLM 生成假设文档进行检索的基线方法

## 关联笔记
- 01KJBZ84FXP9XQZ7489TPV0A5F.md: ChatDBA 笔记中直接引用了 QA-RAG 的 fake answer 召回方法
- 01KJBYZ2DGB2PDS3ADP9JSCQ16.md: HyDE 测试方法笔记，QA-RAG 思路类似于 HyDE 但使用微调 LLM
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记，将 HyDE 归类为输入增强技术中的查询改写方法
