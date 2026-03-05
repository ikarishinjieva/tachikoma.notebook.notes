---
note: 01KJBZ68S0R9NS2HK7MWN06G5A.md
title: 20240227 - 对embedding进行微调
indexed_at: 2026-03-05T09:37:46.459936+00:00
---

## 摘要
对比多种 Embedding 模型（mcontriever+OpenAI、BAAI/llm-embedder、BAAI/bge-large-zh-v1.5、BAAI/bge-m3）在检索任务上的效果。对 BAAI/bge-large-zh-v1.5 进行微调实验，epoch=5 时 loss 从 0.68 降至 0.38，检索效果有所改善。

## 关键概念
- EnsembleRetriever: 将多个 embedding 模型结果结合的检索器
- BAAI/bge-large-zh-v1.5: 智源开源的中文向量嵌入模型
- BAAI/bge-m3: 智源开源的多语言向量嵌入模型
- mcontriever: Facebook 开源的稠密检索模型
- 微调 (Fine-tuning): 在预训练模型基础上用特定数据继续训练

## 关联笔记
- 01KJBZADENS4HHC18MT0EAZ182.md: 提到使用 mcontriever 进行文档召回，与本笔记的 mcontriever+OpenAI 方案相关
- 01KJBZAQZ6CX2KZQ4FC02BPFXS.md: 使用 bge-m3 进行向量化和检索，与本笔记的 bge-m3 测试相关
- 01KJBZ65Y2QFCFWKWQVM7T6NQ3.md: 涉及 FAISS 文档召回方案，与本笔记的 FAISS 导入相关
