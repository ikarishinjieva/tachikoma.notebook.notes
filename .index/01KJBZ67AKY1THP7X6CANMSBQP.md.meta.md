---
note: 01KJBZ67AKY1THP7X6CANMSBQP.md
title: 20240220 - 对reranker进行微调
indexed_at: 2026-03-05T09:37:10.714867+00:00
---

## 摘要
记录使用 FlagEmbedding 对 BGE-reranker-base 进行微调的实验过程，包括训练命令、三元组数据构造方法及多组实验结果对比。发现扩大训练数据集比增加 epoch 或使用 Hard Negatives 效果更显著。

## 关键概念
- 三元组排序 (Triplet Ranking): 使用 (锚点，正样本，负样本) 形式表达文档相关度排序
- Hard Negatives: 使用与正样本相似的负样本进行训练，提升模型区分能力
- 梯度累积 (Gradient Accumulation): 通过多次小批量累积梯度模拟大批量训练
- 微调过拟合: 模型在训练数据上效果好，但对陌生数据泛化能力下降

## 关联笔记
- 01KJBZ65Y2QFCFWKWQVM7T6NQ3.md: TODO 中提及对 BGE Reranker 进行微调的参考链接
