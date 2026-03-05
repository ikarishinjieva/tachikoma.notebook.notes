---
note: 01KJBZ5SKSN1ZNR6HMVAT6JG8H.md
title: 20240209 - 阅读论文: Query2doc: Query Expansion with Large Language Models
indexed_at: 2026-03-05T09:32:38.898385+00:00
---

## 摘要
Query2Doc 利用大语言模型生成伪文档，与原始查询拼接实现查询扩展，弥补查询与文档间的词汇差距。该方法在稀疏检索（BM25 提升 3%-15%）和密集检索器上均取得显著性能提升，尤其对困难查询和领域外数据集效果突出。

## 关键概念
- Query2Doc: 利用 LLM 生成伪文档进行查询扩展的信息检索技术
- 伪文档: 由大语言模型通过 few-shot prompting 生成的与查询相关的假想文档
- 稀疏检索: 基于词汇匹配的信息检索方法（如 BM25）
- 密集检索: 基于嵌入向量的语义检索方法（如 DPR、E5）
- 查询扩展: 通过添加相关信息扩展原始查询以提升检索性能的技术

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文笔记，在检索增强方法部分引用了 Query2Doc
- 01KJBZ5717N16C7ZP3KFQKF32G.md: Modular RAG 综述笔记，将 Query2Doc 列为查询重写的典型方法
- 01KJBZG5NHQZ849FWASZ3XZ40Q.md: Contextual Retrieval 笔记，涉及 BM25+Embeddings 混合检索策略
