---
note: 01KJBZG9HE3MZ0312PBHKXC0M7.md
title: 20241007 - 阅读论文*: Speculative RAG: Enhancing Retrieval Augmented Generation through Drafting
indexed_at: 2026-03-05T10:47:19.945512+00:00
---

## 标签
RAG 优化，文档采样，模型蒸馏，草稿生成，验证机制，推理依据

## 摘要
Speculative RAG 通过聚类采样将召回文档分成多个子集，并行生成答案草稿后择优验证。使用小模型 Drafter 蒸馏大模型的推理依据能力，大模型 Verifier 评估草稿置信度选择最佳答案。

## 关键概念
- Speculative RAG: 通过文档子集并行起草 + 验证的 RAG 优化方法
- Drafter: 经指令微调的小型 LLM，生成答案草稿和推理依据
- Verifier: 大型通用 LLM，评估草稿置信度并选择最佳答案
- 多视角采样: 对检索文档聚类后从每簇采样，减少冗余增加多样性
- 推理依据: LLM 生成答案时的解释性推理过程，用于蒸馏训练 Drafter

## 关联笔记
- 01KJBZG2VSA818F7DC1JG7HF8X.md: EfficientRAG 同样针对 RAG 效率优化，使用轻量级组件降低延迟成本
- 01KJBZG3NMEV9RPF9NN5VBTQ9Z.md: RAGCHECKER 在 claim 级别评估 RAG 系统质量，与本笔记的验证器思路相关
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 结合 MoE 架构改进 RAG，使用反射标记控制检索和答案排序
