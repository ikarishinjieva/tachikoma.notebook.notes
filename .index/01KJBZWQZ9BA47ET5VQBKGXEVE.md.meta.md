---
note: 01KJBZWQZ9BA47ET5VQBKGXEVE.md
title: 20250808 - 分析ms-swift的GRPO compute_loss函数
indexed_at: 2026-03-05T11:51:33.374395+00:00
---

## 摘要
详细分析 ms-swift 中 GRPO 算法的 compute_loss 函数实现，包括 token 级对数概率、熵分位数过滤、KL 散度计算等核心步骤。重点解析 importance_sampling_level 参数如何控制 token 级与序列级的概率修正，以及 PPO-Clip 损失计算的五步实现机制。

## 关键概念
- importance_sampling_level: 控制重要性采样权重按 token 级还是序列级计算，序列级会将 token 对数概率差异平均化
- 熵 (Entropy): 表示模型对 token 的自信程度，概率越均匀熵越大，越不自信
- 对数概率 (logps): 已选中 token 概率的对数值，用于计算新旧策略的概率比率
- coef_1/coef_2: 分别对应 PPO 公式中的概率比 r_t 和裁剪后的概率比 clip(r_t, 1-ε, 1+ε)
- Advantage: 优势值，从 reward 函数得到，用于衡量动作相对于平均水平的优劣

## 关联笔记
- 01KJBZWENK08X8VG36C8Y3AR3F.md: 对 GSPO 的理解，笔记中明确引用了该篇来解释 importance_sampling_level
- 01KJBZVR7G5C69KG304370BJ9D.md: 对 GRPO 的理解，同主题的基础概念笔记
- 01KJBZW8ND6Q8NTEDX7B4WWACP.md: 进行 SQL 优化的 GRPO 实践，包含 GSPO 参数实验和 ms-swift 使用记录
