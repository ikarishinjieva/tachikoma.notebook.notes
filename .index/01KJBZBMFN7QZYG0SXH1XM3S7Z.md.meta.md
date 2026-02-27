---
note: 01KJBZBMFN7QZYG0SXH1XM3S7Z.md
title: 20240725 - LlamaIndex Property Graph Index 分析
indexed_at: 2026-02-27T11:08:52.864621+00:00
---

用户希望我分析这篇关于 LlamaIndex Property Graph Index 的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：LlamaIndex 的 Property Graph Index（属性图索引）功能
2. **核心内容**：
   - 知识图谱增强 RAG 的流程
   - 图谱构建组件（kg_extractors）
   - 检索和查询组件（Retrievers）
   - 存储选项
   - 高级定制

3. **标签候选**：LlamaIndex、知识图谱、RAG、Property Graph、信息提取、向量检索

4. **摘要**：需要 2-3 句话概括

5. **关键概念**：
   - DynamicLLMPathExtractor
   - SimpleLLMPathExtractor
   - TextToCypherRetriever
   - VectorContextRetriever
   - Property Graph Index

6. **关联笔记**：查看其他笔记列表，都是关于 MySQL、Clickhouse、数据库、性能调试相关的内容，与这篇 LlamaIndex/知识图谱/RAG 的笔记没有直接关联。

让我生成输出：
## 标签
LlamaIndex, 知识图谱, RAG, Property Graph, 信息提取, 向量检索

## 摘要
介绍 LlamaIndex 中 Property Graph Index 的完整使用流程，包括图谱构建、检索查询和存储方案。详细对比了多种图谱提取器（kg_extractors）和检索器（Retrievers）的功能差异与适用场景。

## 关键概念
- Property Graph Index: 使用知识图谱增强 RAG 的索引机制
- DynamicLLMPathExtractor: 基于预定义实体/关系类型列表，用 LLM 灵活提取三元组
- SimpleLLMPathExtractor: 使用 LLM 提取简单的主谓宾三元组关系
- VectorContextRetriever: 基于嵌入向量相似度检索图谱节点
- TextToCypherRetriever: 用 LLM 将自然语言查询转换为 Cypher 语句执行

## 关联笔记
无
