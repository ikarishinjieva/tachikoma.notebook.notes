---
note: 01KJBZ5BY0F6897T3ZEDW3GA6H.md
title: 20240204 - 阅读论文: UPRISE: Universal Prompt Retrieval for Improving Zero-Shot Evaluation
indexed_at: 2026-03-05T09:29:45.278267+00:00
---

## 摘要
论文提出 UPRISE 方法，通过训练轻量级检索器自动为零样本任务检索合适的 prompt 模板。检索器在多样化任务上微调，可跨任务、跨模型泛化，在 BLOOM-71B、OPT-66B、GPT3-175B 等大模型上验证有效。

## 关键概念
- 零样本评估 (Zero-Shot Evaluation): 在没有任务特定微调或提示工程的情况下测试模型泛化能力
- Prompt 检索 (Prompt Retrieval): 从预构建的 prompt 池中自动检索最适合给定任务输入的提示模板
- 跨模型泛化 (Cross-Model Generalization): 在小模型 (GPT-Neo-2.7B) 上训练检索器，在大模型上测试
- 提示池 (Prompt Pool): 预先构建的包含多种任务提示的集合，供检索器选择
- 幻觉缓解 (Hallucination Mitigation): UPRISE 能在 ChatGPT 实验中减少模型幻觉问题

## 关联笔记
- 01KJBZ5717N16C7ZP3KFQKF32G.md: RAG 综述笔记，在增强方法章节引用了 UPRISE 作为检索增强的相关技术
