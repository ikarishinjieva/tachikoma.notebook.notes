---
note: 01KJBZY0MAYHH592A9NG68XWV3.md
title: 20250813 - 阅读论文*: (GRPO在小模型上的训练方法) Reinforcement Learning for Reasoning in Small LLMs: What Works and What Doesn’t
indexed_at: 2026-03-05T11:53:02.674838+00:00
---

## 标签
GRPO, 强化学习，小模型，推理能力，训练方法，奖励设计

## 摘要
该论文研究了在严格资源限制下，如何使用 GRPO 强化学习算法提升小模型（1-10B 参数）的推理能力。通过三个递进实验揭示了小模型训练的三大洞见：对高质量数据敏感但易受长度限制影响、混合难易度问题可提升早期稳定性、余弦奖励可有效控制输出长度。

## 关键概念
- GRPO（Group Relative Policy Optimization）: 无需独立评判模型的强化学习算法，显著降低计算开销
- 余弦奖励（Cosine Reward）: 基于输出长度的奖励机制，用于控制模型生成简洁答案


- 高质量数据集：通过多步过滤构建的紧凑、高质量数学推理数据集
- 奖励稀疏问题：小模型训练中因输出过长导致的不稳定现象

## 关联笔记
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 同一天记录的 GRPO 训练实践，涉及 SQL 优化场景中的训练参数调整
- 01KJBZVR7G5C69KG304370BJ9D.md: GRPO 算法的核心机制和训练步骤详解
- 01KJBZW8ND6Q8NTEDX7B4WWACP.md: GRPO 在 SQL 优化中的实际应用，探讨训练效果与校验效果的差异缩小方法## 标签
GRPO, 强化学习，小模型，推理能力，训练方法，奖励设计

## 摘要
该论文研究了在严格资源限制下，如何使用 GRPO 强化学习算法提升小模型（1-10B 参数）的推理能力。通过三个递进实验揭示了小模型训练的三大洞见：对高质量数据敏感但易受长度限制影响、混合难易度问题可提升早期稳定性、余弦奖励可有效控制输出长度。最终模型以仅 42 美元的训练成本实现了与 o1-preview 等顶尖模型相媲美的性能。

## 关键概念
- GRPO（Group Relative Policy Optimization）: 无需独立评判模型的强化学习算法，显著降低计算开销
- 余弦奖励（Cosine Reward）: 基于输出长度的奖励机制，用于控制模型生成简洁答案
- 高质量数据集: 通过多步过滤构建的紧凑、高质量数学推理数据集
- 训练不稳定: 小模型对长度限制敏感，长时间训练可能导致输出混乱

## 关联笔记
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 同一天记录的 GRPO 训练实践，涉及 SQL 优化场景中的训练参数调整
- 01KJBZVR7G5C69KG304370BJ9D.md: GRPO 算法的核心机制和训练步骤详解
- 01KJBZW8ND6Q8NTEDX7B4WWACP.md: GRPO 在 SQL 优化中的实际应用，探讨训练效果与校验效果的差异缩小方法
