---
note: 01KJBZR99GXJ9MHN1GAXC66TH4.md
title: 20250206 - 阅读论文*: Adaptive Note: Retriever-and-Memory: Towards Adaptive Note-Enhanced Retrieval-Augmented Generation
indexed_at: 2026-03-05T11:23:50.334532+00:00
---

## 标签
RAG, 检索增强生成, 自适应笔记, 知识管理, 迭代检索, ARAG

## 摘要
Adaptive Note 是一种通过笔记形式组织和管理检索信息的 RAG 方法，支持 zero-shot 梳理和笔记质量评估。其核心创新在于基于笔记质量变化的自适应停止策略，避免了依赖 LLM 预测检索时机的偏差，同时促进不同检索步骤间的信息整合。

## 关键概念
- Adaptive Note: 以笔记形式采集和整理问题及其衍生问题的信息，支持零样本梳理
- 自适应停止策略: 基于笔记质量变化判断是否停止迭代，而非依赖 LLM 预测
- 知识增长视角: 每次迭代增加新信息并整合到现有知识结构中，避免盲目探索
- ARAG: Agentic Retrieval-Augmented Generation，代理式检索增强生成方法

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 同样结合 RAG 与推理，使用反射标记触发检索，与 Adaptive Note 的迭代检索机制形成对比
- 01KJBZR62TY2F0CMS66NRZXDFS.md: Golden-Retriever 是另一种 Agentic RAG 方案，在增强问题阶段与 Adaptive Note 的信息整合思路有相似之处
- 01KJBZR7NKRXCK2317AES6GZVK.md: RARE 在推理路径中引入 RAG 扩展思维素材，与 Adaptive Note 的笔记增长视角可相互参照
