---
note: 01KJBZ68S0R9NS2HK7MWN06G5A.md
title: 20240227 - 对embedding进行微调
indexed_at: 2026-02-27T10:47:08.522953+00:00
---

用户希望我分析这篇关于 embedding 微调的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：对 embedding 进行微调
2. 主要内容：
   - 使用 embedding 将数据导入 FAISS
   - 评估不同 embedding 模型的效果（mcontriever + openai、BAAI/llm-embedder、BAAI/bge-large-zh-v1.5、BAAI/bge-m3）
   - 对 BAAI/bge-large-zh-v1.5 进行微调，比较不同 epoch 的效果

3. 标签应该是：embedding、微调、FAISS、检索、模型评估

4. 摘要：笔记记录了多种 embedding 模型的评估对比，并对 BAAI/bge-large-zh-v1.5 进行微调实验，通过不同 epoch 的训练验证效果改善。

5. 关键概念：
   - EnsembleRetriever: 结合多个 embedding 的检索器
   - FAISS: 向量相似度搜索库
   - embedding 微调: 对预训练向量模型进行针对性训练
   - epoch: 训练轮次

6. 关联笔记：从提供的笔记列表来看，这些笔记大多是关于 MySQL、Clickhouse、系统性能等方面的，与 embedding/向量检索没有直接关联，所以应该是「无」

让我按照格式输出。
## 标签
embedding, 微调, FAISS, 向量检索, 模型评估, BGE

## 摘要
笔记记录了多种 embedding 模型（mcontriever+openai、llm-embedder、bge-large-zh-v1.5、bge-m3）的效果对比评估。通过对 BAAI/bge-large-zh-v1.5 进行微调实验，验证了训练后效果有所改善。

## 关键概念
- EnsembleRetriever: 将多个 embedding 模型结合使用的检索器
- FAISS: Facebook 开源的向量相似度搜索库
- embedding 微调: 对预训练向量模型使用特定数据进行再训练
- epoch: 模型训练的完整轮次

## 关联笔记
无
