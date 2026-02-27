---
note: 01KJBYXSJ81X1F9GDQ6P2VX15P.md
title: 20230409 - llama-index 逻辑整理
indexed_at: 2026-02-27T10:25:45.411946+00:00
---

用户要求我分析这篇关于 llama-index 逻辑整理的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：llama-index 框架的核心逻辑流程，包括索引构建和查询两个主要部分

2. **核心内容**：
   - BaseGPTIndex.from_documents：从文档生成索引的流程
   - BaseGPTIndex.query：查询流程
   - 涉及各种索引类型（GPTVectorStoreIndex、GPTListIndex 等）
   - 涉及 embedding、node 分割、query 处理等

3. **标签**：应该是 LLM、索引、RAG、embedding、查询 等

4. **摘要**：需要概括笔记核心内容

5. **关键概念**：BaseGPTIndex、Node Parser、Embedding、Query Runner、Vector Store 等

6. **关联笔记**：查看提供的笔记列表，这些都是关于数据库、MySQL、Clickhouse、系统性能等的笔记，与 llama-index/LLM 索引没有直接关联，所以应该写「无」

让我生成输出：
## 标签
LLM, 索引构建, 向量检索, RAG, Embedding, 查询流程

## 摘要
本笔记整理了 llama-index 框架的核心逻辑，包括从文档构建索引（from_documents）和执行查询（query）的完整流程。涵盖了节点解析、向量化、索引存储及查询响应生成等关键步骤。

## 关键概念
- BaseGPTIndex: 索引基类，提供 from_documents 构建和 query 查询的核心接口
- Node Parser: 将文档切割成文本片段并生成节点对象
- Embedding: 将文本转换为向量表示，用于相似度检索
- Query Runner: 处理查询请求，包括查询转换和执行器分发
- Vector Store: 存储向量索引并支持基于嵌入的 top-k 检索

## 关联笔记
无
