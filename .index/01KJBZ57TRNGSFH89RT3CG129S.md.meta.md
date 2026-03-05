---
note: 01KJBZ57TRNGSFH89RT3CG129S.md
title: 20240203 - 阅读论文: Query Rewriting for Retrieval-Augmented Large Language
indexed_at: 2026-03-05T09:29:06.018331+00:00
---

## 摘要
论文提出"Rewrite-Retrieve-Read"流程，通过查询重写改进检索增强语言模型的性能。使用伪数据预热结合强化学习 (PPO) 训练可训练重写器，解决无对口训练数据的问题。实验证明该方法在开放领域问答任务中优于直接检索和 LLM 重写。

## 关键概念
- Rewrite-Retrieve-Read: 三步骤流程，先重写查询、再检索、最后阅读生成答案
- 伪数据预热: 用 LLM 生成查询作为伪标签构建训练数据集，筛选正确预测样本
- 强化学习 (PPO): 以 LLM 阅读器反馈为奖励信号，进一步优化重写器适应下游任务
- 可训练重写器: 基于 T5-large 的小型语言模型，接管查询重写步骤并与冻结模块对齐

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 使用反射标记控制检索时机，与本文重写查询优化检索目的一致
- 01KJBZG2VSA818F7DC1JG7HF8X.md: EfficientRAG 迭代生成新查询进行检索，方法不同但目标相似
- 01KJBZRB8197X3KFWB1F72P7VG.md: RICHES 在生成过程中限制检索词必须出现在语料库中，与查询重写思路相关
