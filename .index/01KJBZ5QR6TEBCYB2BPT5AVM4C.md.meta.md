---
note: 01KJBZ5QR6TEBCYB2BPT5AVM4C.md
title: 20240208 - 阅读论文: SURGE: Knowledge Graph-Augmented Language Models for Knowledge-Grounded Dialogue ...
indexed_at: 2026-03-05T09:32:17.167969+00:00
---

## 标签
SURGE, 知识图谱, 对话生成, 图文对比学习, 子图检索, RAG

## 摘要
笔记记录了 SURGE 论文的阅读问答，该框架使用知识图谱增强对话生成，包含上下文相关子图检索、子图不变编码和图文对比学习三个核心步骤。最终结论认为知识图谱应用算法过于复杂，暂时搁置此论文。

## 关键概念
- SURGE 框架: 知识图谱增强的对话生成框架，通过检索子图并编码来保持生成一致性
- 上下文相关子图检索: 从知识图谱中检索与对话历史相关的子图获取相关知识
- 子图不变编码: 将子图结构信息转化为固定长度向量表示供神经网络处理
- 图文对比学习: 多模态对比学习方法，确保生成文本与检索知识在语义空间保持一致

## 关联笔记
- 01KJBZ5717N16C7ZP3KFQKF32G.md: RAG 综述论文笔记，其中第 6 章提到了 SURGE 框架
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 论文笔记，同样使用知识图谱进行检索增强生成
- 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md: 知识图谱关系抽取实践笔记，探索将文档转换为知识图谱的方法
