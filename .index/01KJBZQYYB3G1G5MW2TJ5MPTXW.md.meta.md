---
note: 01KJBZQYYB3G1G5MW2TJ5MPTXW.md
title: 20250131 - 阅读论文: CPO: Chain of Preference Optimization: Improving Chain-of-Thought Reasoning in LLMs
indexed_at: 2026-03-05T11:17:23.794783+00:00
---

## 摘要
CPO 利用 ToT 搜索过程中产生的偏好信息，通过 DPO 算法训练 LLM 在 CoT 推理中选择更优步骤。与 FPO 不同，CPO 在链级进行优化，避免了 LCP 梯度消失问题。

## 关键概念
- CPO (Chain of Preference Optimization): 通过 DPO 训练模型学习 CoT 推理中每一步的偏好选择
- ToT (Tree of Thoughts): 探索多种推理路径并评估每一步候选思维，为 CPO 提供训练数据
- DPO (Direct Preference Optimization): 直接基于偏好对优化模型的离线 RL 方法
- 链级优化: 针对推理链中每一步进行优化，而非仅关注最终答案
- FPO (Full-path Preference Optimization): 仅关注最终答案的偏好优化方法，易出现梯度消失问题

## 关联笔记
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: 介绍 DPO 和 GRPO 在对齐中的应用，DPO 是 CPO 的基础训练方法
- 01KJBZR1YB6S9NAWE04BZY5WY3.md: 讨论 TPO 如何用 DPO 训练思考过程，与 CPO 理念相似
- 01KJBZRA5C93JENTBEKNWCFPTQ.md: 分析 DPO 结合 NLL 训练的作用，涉及偏好优化的训练细节
