---
note: 01KJBZFC97Q7JPHD2GFX6VGG92.md
title: 20240914 - 使用 Llama-factory 做强化学习
indexed_at: 2026-03-05T10:41:53.886691+00:00
---

## 标签
Llama-factory, 强化学习，PPO, Reward Model, LoRA, Qwen2

## 摘要
记录使用 Llama-factory 进行强化学习（PPO）的完整流程，包括环境配置、安装步骤和训练命令。涵盖 Reward Model 训练、数据集配置及数据集加载慢等问题的排查方案。

## 关键概念
- PPO: 近端策略优化算法，用于强化学习训练
- Reward Model: 奖励模型，用于评估生成回答的质量
- LoRA: 低秩适配器，参数高效的微调方法
- 偏好数据集: 用于训练 Reward Model 的成对偏好数据

## 关联笔记
- 01KJBZFPYRXEB4AXWM46JQK880.md: 同一系列的后续笔记，继续记录 Llama-factory 强化学习的进阶实验
- 01KJBZN4KQV7NK5GRT46165QTR.md: 总结 Llama-factory 存在 bug，推荐使用 ms-swift 的后续知识整理
- 01KJBZFBKFBR5T4W3B4JS82C1N.md: 前期评估笔记，提到下一步用 Llama-factory 尝试 sft+ppo 的计划
