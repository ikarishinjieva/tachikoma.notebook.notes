---
note: 01KJBZ0NDKWT4C87MXDZ2QZS0K.md
title: 20230530 - 大模型切分算法优化
indexed_at: 2026-03-05T08:46:31.558278+00:00
---

## 摘要
探讨本地知识库文档切分策略以保留语义完整性，包括段落切分、关键句提取、实体中心切分等方法。对比多种关键字提取算法（TF-IDF、TextRank、RAKE、LDA），并测试 contextual compression 和 reranker 等检索优化技术。

## 关键概念
- TF-IDF: 通过词频和逆文档频率计算词语重要性
- TextRank: 基于 PageRank 算法的句子/短语重要性排序
- Contextual Compression: 筛选文档切片中与问题相关的部分
- 语义分割: 使用 NLP 模型将文档分成语义一致的片段
- RAKE: 基于词频和共现关系的快速关键字提取方法

## 关联笔记
- 01KJBZFPM6BX07HW9FQM0QZ670.md: dsRAG 项目对 chunking 的创新，包含语义切片和相关区段提取技术
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 的 RAG 方案，包含自主关键词提取和多步检索流程
