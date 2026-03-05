---
note: 01KJBZ9T3AEMRFET8C9N2T38NT.md
title: 20240611 - 阅读论文: ACTIVERAG: Revealing the Treasures of Knowledge via Active Learning
indexed_at: 2026-03-05T10:04:30.920006+00:00
---

## 摘要
ActiveRAG 提出四种认知代理（语义关联、认知锚定、逻辑推理、认知校准）从不同维度分析检索内容，通过认知连接机制将多代理分析结果整合到思维链中。该方法旨在提升 RAG 系统对知识的深度理解和推理能力，减少模型幻觉。

## 关键概念
- 认知连接 (Cognitive Nexus): 将四种代理的分析结果整合到思维链中生成最终答案的机制
- 语义关联代理 (Associate): 将新信息与 LLM 已有知识网络建立语义联系
- 认知锚定代理 (Anchoring): 识别并解释模型未知的概念和信息
- 逻辑推理代理 (Logician): 从检索内容中提取结构化信息进行逻辑推理
- 认知校准代理 (Cognition): 识别并解决新信息与模型现有知识之间的矛盾

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 同样结合 RAG 与推理能力，使用反射标记实现检索增强推理
- 01KJBZRB8197X3KFWB1F72P7VG.md: From RAG to RICHES 探讨检索与序列生成交织的 RAG 改进方案
- 01KJBZRDJC48S1985E9C14H3W2.md: RAGCHECKER 提供 RAG 系统的细粒度评估框架，可用于验证 ActiveRAG 效果
