---
note: 01KJBZ9Q7WJ031P12NZ4NDBKRF.md
title: 20240610 - 阅读论文: GRG: Generator-Retriever-Generator Approach for Open-Domain Question Answering
indexed_at: 2026-03-05T10:01:08.128566+00:00
---

## 摘要
GRG 是一种三阶段开放域问答框架，结合 InstructGPT 的上下文生成能力与双编码器的文档检索能力。最终将问题、检索文档和生成上下文整合后输入 LLaMA 生成答案，旨在利用外部知识库和生成新信息实现更全面的问答。

## 关键概念
- GRG (Generator-Retriever-Generator): 三阶段问答框架，依次进行文档生成、文档检索、答案生成
- 双编码器网络：分别将问题和文档编码为向量，通过点积或余弦相似度计算相关性
- 检索 - 阅读 vs 检索 - 生成：前者用阅读理解模型提取答案片段，后者用序列到序列模型生成答案
- 上下文生成：使用 InstructGPT 根据问题生成潜在答案上下文作为补充信息

## 关联笔记
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 笔记涉及微调+RAG 的医疗问答方案，与 GRG 同属检索增强生成领域
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 笔记介绍结合检索与生成的推理架构，与 GRG 都探索检索 - 生成协同
- 01KJBZG3NMEV9RPF9NN5VBTQ9Z.md: RAGCHECKER 笔记提供评估检索器和生成器的细粒度框架，可用于诊断 GRG 性能
