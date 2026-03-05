---
note: 01KJBZR0R965XV1E3R1YF93FBG.md
title: 20250201 - 阅读论文*: DR-RAG: Applying Dynamic Document Relevance to RetrievalAugmented Generation for Question-Answering
indexed_at: 2026-03-05T11:18:30.351352+00:00
---

## 摘要
DR-RAG 通过两阶段检索解决"对回答有贡献的文档与问题文本相关性低"的问题：先用传统方法召回文档，再将问题与召回文档拼接成新查询进行二次召回。使用分类器筛选候选文档，通过正向选择 (CFS) 和反向选择 (CIS) 两种策略减少冗余，最终将筛选后的文档送入 LLM 生成答案。

## 关键概念
- 两阶段检索: 第一次按相似度召回文档，第二次将问题与召回文档拼接成新查询再次召回
- 查询文档拼接 (QDC): 将原始问题与第一阶段召回的文档拼接构成新查询，需考虑语义组合而非简单字符串拼接
- 正向选择 (CFS): 对每个第一阶段文档，选择第一个被分类器判断为有贡献的第二阶段文档
- 反向选择 (CIS): 将与所有其他文档组合都被判断为负面的文档视为冗余并移除

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 同样采用多阶段检索策略，使用反射标记触发检索并结合 MoE 架构
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 也关注减少冗余文档，通过聚类采样和草稿验证优化召回文档质量
- 01KJBZRDJC48S1985E9C14H3W2.md: RAGCHECKER 提供细粒度评估框架，可用于诊断 DR-RAG 等 RAG 系统的检索器和生成器质量
