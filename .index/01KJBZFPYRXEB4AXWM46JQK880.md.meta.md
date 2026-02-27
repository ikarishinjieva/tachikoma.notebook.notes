---
note: 01KJBZFPYRXEB4AXWM46JQK880.md
title: 20240926 - 使用 Llama-factory 做强化学习 [2]
indexed_at: 2026-02-27T11:16:08.438264+00:00
---

用户要求我分析这篇关于使用 Llama-factory 做强化学习的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：使用 Llama-factory 进行强化学习训练，包括 reward model 训练、SFT 微调和 PPO 训练
2. **核心技术**：Llama-factory、LoRA 微调、Reward Model、PPO、Qwen2-7B
3. **数据集**：chinese-enthusiastic-dpo、chinese-medqa

现在生成各个部分：

**标签**：需要 3-6 个关键词
- Llama-factory
- 强化学习
- PPO
- Reward Model
- LoRA 微调
- Qwen2

**摘要**：2-3 句话概括
记录使用 Llama-factory 框架对 Qwen2-7B 模型进行强化学习训练的过程，包括 Reward Model 训练、SFT 微调和 PPO 训练三个阶段的配置与结果。Reward Model 在 0.11 轮次准确率达 0.988，但可能存在过拟合风险。

**关键概念**：3-5 个
- Reward Model: 用于评估模型输出质量的打分模型
- PPO (Proximal Policy Optimization): 强化学习算法，用于优化策略
- LoRA: 低秩适配器微调技术，减少训练参数量
- SFT (Supervised Fine-Tuning): 监督微调阶段


- Llama-factory: 大模型微调框架，支持多种训练方式

**关联笔记**：查看提供的笔记列表，这些都是 2021 年的数据库、系统性能相关的笔记，与当前的 LLM 强化学习内容没有直接关联，应该标注为无。## 标签
Llama-factory, 强化学习, PPO, Reward Model, LoRA 微调, Qwen2

## 摘要
记录使用 Llama-factory 框架对 Qwen2-7B 模型进行强化学习训练的全过程，涵盖 Reward Model 训练、SFT 微调和 PPO 训练三个阶段。Reward Model 在 0.11 轮次时准确率达 0.988，但可能存在过拟合风险，未测试泛化性。

## 关键概念
- Reward Model: 用于评估模型输出质量的打分模型，在 DPO/PPO 中提供奖励信号
- PPO (Proximal Policy Optimization): 强化学习算法，通过策略梯度优化模型输出
- LoRA: 低秩适配器微调技术，仅训练少量参数实现高效微调
- SFT (Supervised Fine-Tuning): 监督微调阶段，使用标注数据训练模型
- Llama-factory: 一站式大语言模型微调框架，支持多种训练阶段

## 关联笔记
无
