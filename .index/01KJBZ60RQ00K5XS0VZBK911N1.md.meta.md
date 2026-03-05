---
note: 01KJBZ60RQ00K5XS0VZBK911N1.md
title: 20240210 - 阅读论文: RRR: Query rewriting for retrieval-augmented large language models
indexed_at: 2026-03-05T09:34:56.444893+00:00
---

## 摘要
论文提出 Rewrite-Retrieve-Read(RRR) 框架，通过查询重写优化检索增强大语言模型的检索查询质量。引入可训练重写器，采用预热 + 强化学习两阶段训练方案，使小型模型能为黑盒 LLM 生成更适配的检索查询。

## 关键概念
- Rewrite-Retrieve-Read: 先重写查询、再检索文档、最后阅读生成的三阶段框架
- 查询重写: 用 LLM 将原始问题改写为更适合搜索引擎的查询形式
- 可训练重写器: 小型语言模型替代黑盒 LLM 执行查询重写，支持端到端优化
- 预热训练: 用 LLM 生成的伪数据预训练重写器，筛选正确样本构建训练集
- 强化学习: 用 PPO 算法根据阅读器反馈优化重写器，适应下游任务

## 关联笔记
- 01KJBZ57TRNGSFH89RT3CG129S.md: 同一篇论文的另一篇笔记，详细记录了 Trainable Scheme 和强化学习训练细节
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，在检索增强分类中引用了 RRR 框架
- 01KJBZ5717N16C7ZP3KFQKF32G.md: RAG 综述笔记，将 Rewrite-Retrieve-Read 归类为 Modular RAG 的典型模式
