---
note: 01KJBYYSWT0HJ1ZYSW82YFETXD.md
title: 20230429 - 文章解析: LLM的知识获取架构
indexed_at: 2026-03-05T08:39:37.724768+00:00
---

## 摘要
解析 LLM 知识获取架构文章，对比上下文注入与微调两种方法的局限性，介绍 llama_index 的检索增强方案。重点讲解 Hypothetical Document Embeddings (HyDE) 和 Dense Retriever (Contriever) 的技术原理与实现细节。

## 关键概念
- Hypothetical Document Embeddings (HyDE): 对问题生成虚假回答，然后在文档库中查询与虚假回答相似的真實信息
- Dense Retriever: 基于深度学习的信息检索方法，将文档和查询映射到高维向量空间进行语义相似度匹配
- Contriever: Facebook 开发的上下文密集检索器，对文档库和查询使用不同的分词器和编码器
- llama_index: 用于构建 LLM 应用的索引和检索框架，提供多种索引结构和检索策略

## 关联笔记
- 01KJBYZ2DGB2PDS3ADP9JSCQ16.md: 实际测试 Hypothetical Document Embeddings (HyDE) 方法的实现效果
- 01KJBZ5717N16C7ZP3KFQKF32G.md: RAG 综述论文笔记，涵盖检索器、生成器等核心概念
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: llama_index 的 response_mode 和 refine 机制使用细节
