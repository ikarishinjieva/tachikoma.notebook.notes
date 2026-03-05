---
note: 01KJBZFPYRXEB4AXWM46JQK880.md
title: 20240926 - 使用 Llama-factory 做强化学习 [2]
indexed_at: 2026-03-05T10:42:42.155223+00:00
---

## 摘要
记录使用 Llama-factory 进行 Reward Model 训练和 PPO 强化学习的全过程。包含 Reward Model 训练配置（准确率 0.988）、SFT 微调对比、PPO 训练多次迭代实验及问题排查（LoRA 权重全零、输出为空等）。

## 关键概念
- Reward Model: 用于 PPO 训练中评估生成内容质量的判别模型
- PPO (Proximal Policy Optimization): 近端策略优化，一种稳定的强化学习算法
- LoRA: 低秩适配器，用于参数高效的模型微调
- SFT (Supervised Fine-Tuning): 监督微调，PPO 训练前的初始化步骤

## 关联笔记
- 01KJBZFC97Q7JPHD2GFX6VGG92.md: 同一系列前篇，记录 20240914 首次使用 Llama-factory 做强化学习的环境搭建和初始尝试
