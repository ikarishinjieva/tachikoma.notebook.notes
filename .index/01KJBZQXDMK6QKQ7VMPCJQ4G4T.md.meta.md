---
note: 01KJBZQXDMK6QKQ7VMPCJQ4G4T.md
title: 20250130 - "拒绝采样"的概念
indexed_at: 2026-03-05T11:16:21.849425+00:00
---

## 摘要
介绍 Llama Post-training 的飞轮迭代过程：通过偏好样本训练 Reward Model，再用其对模型生成采样进行打分筛选，选取高分样本用于 SFT 训练，最后通过 DPO 对齐得到更优模型。强调拒绝采样是自动化样本工程的核心方法，Reward Model 是关键角色。

## 关键概念
- 拒绝采样：用 Reward Model 对模型生成的多个采样结果打分，选取 topN 高分样本作为训练数据
- Reward Model：用于评估生成样本质量的打分模型，是拒绝采样的核心组件
- Post-training 飞轮：通过 SFT→DPO→新模型的迭代循环持续优化模型性能
- SFT Model：使用拒绝采样筛选出的高质量样本进行监督微调的模型
- DPO：直接偏好优化，用于在 SFT 后进行对齐学习

## 关联笔记
- 01KJBZSKZ0GFCFP50QGNNNFB3Y.md: PROMPTCOT 论文也使用拒绝采样方法筛选高质量数学问题数据
- 01KJBZRQ6GE4B51ZFRHXA7TN44.md: 对比研究 SFT 与 RL 在 Post-training 中的泛化能力差异
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: Qwen-Image 采用 SFT→DPO→GRPO 的对齐流程，与 Llama 飞轮类似
