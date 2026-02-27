---
note: 01KJBZ67AKY1THP7X6CANMSBQP.md
title: 20240220 - 对reranker进行微调
indexed_at: 2026-02-27T10:46:41.591521+00:00
---

用户要求我分析一篇关于 reranker 微调的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主题：对 reranker 模型进行微调
2. 主要内容：
   - 在 gpu888.cn 机器上进行 BGE reranker 的微调
   - 训练命令和参数
   - 训练数据格式（三元组方式表达相关度排序）
   - 不同实验结果（epoch、Hard Negatives 等）
   - 后续 TODO

3. 标签应该包括：reranker、微调、FlagEmbedding、训练数据、排序等

4. 摘要：概括微调过程和实验结果

5. 关键概念：reranker、三元组排序、Hard Negatives、微调等

6. 关联笔记：查看提供的笔记列表，这些都是关于 MySQL、Clickhouse、数据库相关的笔记，与 reranker 微调没有直接关联，所以应该是「无」

让我按照格式输出。## 标签
reranker, 微调, FlagEmbedding, 训练数据, 排序模型, 三元组

## 摘要
记录使用 FlagEmbedding 对 BGE reranker 模型进行微调的过程，包括训练命令、数据格式（三元组表达相关度排序）及多轮实验结果。实验表明扩大 epoch 和 Hard Negatives 收效有限，需更多训练数据来提升陌生数据上的效果。

## 关键概念
- 三元组排序 (Triplet Ranking): 使用锚点 (问题)、正样本 (更相关文档)、负样本 (较少相关文档) 表达相关度排序
- Hard Negatives: 选择与正样本相似但实际不相关的负样本进行训练的技术
- Reranker 微调: 对预训练排序模型进行领域适配训练以提升排序效果

## 关联笔记
无
