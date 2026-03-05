---
note: 01KJBZRB8197X3KFWB1F72P7VG.md
title: 20250207 - 阅读论文*: From RAG to RICHES: Retrieval Interlaced with Sequence Generation
indexed_at: 2026-03-05T11:26:08.495921+00:00
---

## 标签
RICHES, RAG, 检索增强生成, FM 索引, 解码器修改, token 选择

## 摘要
RICHES 将检索与序列生成交织，通过修改 decoder 的 argmax 步骤，要求生成的检索词前缀必须存在于语料库中。核心挑战包括与 FM 索引交互、概率分布调整、缓存与回溯机制等工程实现问题。

## 关键概念
- RICHES: Retrieval Interlaced with Sequence Generation，检索与序列生成交织的 RAG 架构
- FM 索引: 全文索引结构，用于快速查询 token 前缀在语料库中的出现情况
- 前缀匹配约束: 解码时强制生成的 token 前缀必须存在于语料库中，否则概率置零
- 概率融合: 将 LLM 自身概率分布与 FM 索引返回的分数进行加权整合

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 同样改进 RAG 架构，使用反射标记控制检索与 MoE 路由机制
- 01KJBZR99GXJ9MHN1GAXC66TH4.md: Adaptive-Note 提出笔记增强检索，通过笔记质量评估实现自适应停止策略
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 在医疗领域应用 RAG，设计自主关键词提取与结构化提示链
