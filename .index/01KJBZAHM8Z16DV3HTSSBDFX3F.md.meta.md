---
note: 01KJBZAHM8Z16DV3HTSSBDFX3F.md
title: 20240710 - 阅读论文: GraphRAG: From Local to Global: A Graph RAG Approach to Query-Focused Summarization
indexed_at: 2026-03-05T10:19:07.278878+00:00
---

## 摘要
GraphRAG 通过将文本转换为实体关系图，利用 Leiden 算法检测层次化社区结构，解决传统 RAG 难以回答全局性问题的局限。核心流程包括图索引构建、社区检测和多层级社区摘要生成，支持从局部到全局的多粒度查询。

## 关键概念
- GraphRAG: 基于图结构的检索增强生成方法，通过实体关系图组织信息
- 社区检测: 使用 Leiden 算法识别图中密集互连的节点集群
- 层次化社区: 不同粒度级别的社区划分，支持多尺度摘要
- 查询焦点摘要 (QFS): 从大量文本中提取与查询相关的内容并生成简洁摘要
- 图索引: 将文本块中的实体、关系、协变量提取后构建的图结构

## 关联笔记
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 同样使用知识图谱增强 RAG，采用 PageRank 算法进行多跳推理
- 01KJBZF7F960C9BRB1RD259XZM.md: 分析 Graph 与 RAG 结合的应用场景，讨论 GraphRAG 适合的问题类型
- 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md: 知识图谱关系抽取实践，探索用知识图谱辅助 RAG 召回
