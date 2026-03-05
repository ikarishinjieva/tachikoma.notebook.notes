---
note: 01KJBZ9DMR1QAYQZRVZ0CVSHG1.md
title: 20240526 - 阅读论文: RaFe: Ranking Feedback Improves Query Rewriting for RAG
indexed_at: 2026-03-05T09:56:31.284931+00:00
---

## 摘要
RaFe 论文提出通过多次查询重写生成候选，利用文档召回和 reranking 排序结果，使用 Precision@K 或 MRR 评估排序质量来优化重写效果。论文区分了在线反馈（PPO 等强化学习）和离线反馈（DPO 等偏好学习）两种训练方式。

## 关键概念
- Precision@K: 前 K 个检索结果中相关文档所占比例，衡量整体相关性
- MRR: 第一个相关文档排名倒数的平均值，关注快速找到最相关结果的能力
- DPO: 基于偏好对的直接优化算法，适用于离线反馈训练
- PPO: 近端策略优化强化学习算法，适用于在线反馈和序列决策
- 查询重写: 通过改写原始查询来提升 RAG 检索效果的技术

## 关联笔记
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 讨论 GRPO 和 DPO 在 SQL 优化训练中的应用
- 01KJBZSH34T3S96WRRSDA6EVVJ.md: 使用 DPO 让模型遵循改写指令的实践
- 01KJBZWENK08X8VG36C8Y3AR3F.md: 分析 GRPO 和 PPO 算法的原理与对比
