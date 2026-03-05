---
note: 01KJBZEC93707PS3CG7V9ET3PX.md
title: 20240815 - 阅读论文*: LARGE LANGUAGE MODELS ARE HUMAN-LEVEL PROMPT ENGINEERS
indexed_at: 2026-03-05T10:33:13.469079+00:00
---

## 摘要
论文提出 APE（Automatic Prompt Engineer）框架，通过正向/反向生成模式产生候选提示词，并结合 beam search 寻找最优解。采用语义相似度重采样进行迭代优化，同时通过多策略避免陷入局部最优。

## 关键概念
- 正向生成 (Forward Generation): 将样本和引导文字拼接成完整 prompt，让 LLM 从头生成指令
- 反向生成 (Reverse Generation): 在 prompt 中预留空位，让 LLM 填空补全指令
- 语义相似度重采样: 生成与已有指令语义相近但表述不同的变体指令
- Beam Search: 用于在候选提示词空间中搜索最优提示词的算法

## 关联笔记
- 01KJBZER5YNJ7WJPHZJPQZCJYV.md: 综述文章提及 APE 作为自动生成和选择指令的方法
- 01KJBZEFVYBDN8ZJD2E8A25KB3.md: 同属提示词工程领域，提出提示词设计模式目录
