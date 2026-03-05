---
note: 01KJBZ5717N16C7ZP3KFQKF32G.md
title: 20240202 - 阅读论文: 综述: Retrieval-Augmented Generation for Large Language Models
indexed_at: 2026-03-05T09:28:44.520844+00:00
---

## 标签
RAG, 检索增强生成, 大语言模型, 论文综述, Naive RAG, Advanced RAG

## 摘要
综述 RAG 技术的完整研究框架，涵盖从 Naive RAG 到 Advanced RAG 再到 Modular RAG 的范式演进。详细对比 RAG 与微调的优劣，并深入分析检索器、生成器、增强方法和评估系统四大核心组件。

## 关键概念
- Naive RAG: 早期 RAG 范式，通过检索 - 生成流程提高回答质量，但存在索引和检索精度问题
- Advanced RAG: 引入预检索和后检索优化，采用滑动窗口、细粒度分割、元数据等技术改进索引质量
- Modular RAG: 将 RAG 系统模块化设计，各模块负责不同任务，提升灵活性和可扩展性
- REALM/RETRO: 在预训练阶段引入检索机制的技术，用于增强开放域问答性能

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 论文笔记，结合 RAG 与 MoE 架构的进阶研究
- 01KJBZR62TY2F0CMS66NRZXDFS.md: Golden-Retriever 笔记，工业级 RAG 系统中术语查询增强的实践方案
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 笔记，医疗领域微调+RAG 方案的深度优化实践
