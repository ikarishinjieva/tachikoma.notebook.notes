---
note: 01KJBZBMFN7QZYG0SXH1XM3S7Z.md
title: 20240725 - LlamaIndex Property Graph Index 分析
indexed_at: 2026-03-05T10:26:30.305199+00:00
---

## 标签
LlamaIndex, Property Graph, 知识图谱，RAG, 图谱检索，信息抽取

## 摘要
分析 LlamaIndex Property Graph Index 模块的完整工作流程，包括图谱构建组件（kg_extractors）和检索器（Retrievers）的功能差异与适用场景。详细介绍了 SimpleLLMPathExtractor、DynamicLLMPathExtractor 等提取器及 LLMSynonymRetriever、TextToCypherRetriever 等检索器的工作机制。

## 关键概念
- Property Graph Index: LlamaIndex 中使用知识图谱增强 RAG 的索引模块
- kg_extractors: 从文本中提取三元组关系的组件，支持多种提取策略
- TextToCypherRetriever: 使用 LLM 将自然语言查询转换为 Cypher 语句的检索器
- SchemaLLMPathExtractor: 基于严格预定义模式提取关系的组件，使用 pydantic 验证

## 关联笔记
- 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md: 同主题讨论知识图谱关系抽取，探索将文档转换为知识图谱辅助 RAG
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 论文分析，同样使用知识图谱增强检索，采用无模式图谱+PageRank 算法
- 01KJBZF7F960C9BRB1RD259XZM.md: Graph 与 RAG 结合的探索，讨论知识图谱适合解决的问题场景
