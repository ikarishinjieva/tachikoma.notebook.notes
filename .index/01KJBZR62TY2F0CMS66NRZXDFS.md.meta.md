---
note: 01KJBZR62TY2F0CMS66NRZXDFS.md
title: 20250203 - 阅读论文: Golden-Retriever: High-Fidelity Agentic Retrieval Augmented Generation for Industrial Knowledge Base
indexed_at: 2026-03-05T11:21:31.843514+00:00
---

## 摘要
Golden-Retriever 针对工业知识库中领域术语和缩写词导致检索困难的问题，提出术语查询解释装置。通过识别术语、识别上下文、查询术语词典、增强问题四个步骤，在在线处理阶段进行基于反思的问题增强，消除歧义以提升 RAG 检索准确性和问答质量。

## 关键概念
- 术语识别 (Identify Jargons): 使用 LLM 识别用户问题中的领域特定术语和缩写，包括拼写错误或未录入的新术语
- 上下文识别 (Identify Context): 通过预定义上下文列表引导 LLM 判断问题所属领域，确保术语正确理解
- 术语查询 (Query Jargons): 使用 SQL 查询预先构建的术语词典获取术语定义和注释，避免 LLM 生成 SQL 的安全风险
- 问题增强 (Augment Question): 将原始问题、上下文信息和术语定义整合为增强问题，消除歧义以提升检索相关性

## 关联笔记
- 01KJBZRB8197X3KFWB1F72P7VG.md: 同属检索词生成优化研究，RICHES 通过约束解码让 LLM 生成语料库中存在的检索词
- 01KJBZR99GXJ9MHN1GAXC66TH4.md: 同为增强型 RAG 研究，Adaptive-Note 使用笔记机制整合多步检索信息并动态评估质量
- 01KJBZRDJC48S1985E9C14H3W2.md: RAGCHECKER 提供细粒度评估框架，可用于诊断 Golden-Retriever 的检索器和生成器性能
