---
note: 01KJBZJCR5P60FXY3BH47TAFH4.md
title: 20241105 - 阅读论文*: HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models
indexed_at: 2026-03-05T10:54:13.093702+00:00
---

## 摘要
HippoRAG 提出了一种基于知识图谱的检索方法，通过无模式图谱构建和 PPR 算法实现高效的多跳推理。核心创新包括知识图谱引导的段落检索、节点特异性权重和概率传播机制，摆脱了传统检索对路径模式和关键词的依赖。

## 关键概念
- 无模式知识图谱：无需预定义 schema，灵活处理各种关系类型，便于整合新知识
- Personalized PageRank (PPR)：基于随机游走的图搜索算法，通过概率传播发现多跳连接
- 节点特异性：类似 IDF，计算公式为|Pi|^(-1)，衡量节点在语料库中的重要性
- 多跳推理：通过图谱结构和概率传播发现隐含的逻辑连接，无需预定义路径

## 关联笔记
- 01KJBZF7F960C9BRB1RD259XZM.md: 讨论 Graph 与 RAG 结合的场景，与 HippoRAG 的图谱检索思路相关
- 01KJBZG2VSA818F7DC1JG7HF8X.md: EfficientRAG 同样针对多跳问题，但采用迭代检索而非图谱方法
- 01KJBZBNS3200SZ8EH9C94S1ZB.md: 知识图谱增强 RAG 的早期探索，与 HippoRAG 的图谱构建思路有共通之处
