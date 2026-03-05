---
note: 01KJBZRA5C93JENTBEKNWCFPTQ.md
title: 20250205 - 阅读论文*: SELF-CONSISTENCY PREFERENCE OPTIMIZATION
indexed_at: 2026-03-05T11:24:56.727122+00:00
---

## 摘要
论文提出基于自我一致性的偏好优化方法：模型对相似问题的多个回答中，一致性越高的答案部分正确率越高。训练时结合 DPO 和 NLL 损失函数，DPO 学习偏好排序，NLL 保持生成文本的自然语言流畅性。

## 关键概念
- SELF-CONSISTENCY PREFERENCE OPTIMIZATION: 利用模型回答的一致性程度构建 DPO 训练对的偏好优化方法
- DPO (Direct Preference Optimization): 通过偏好对比对直接优化模型，拉近与优选答案距离、推远与劣选答案距离
- NLL (Negative Log-Likelihood): 负对数似然损失，用于惩罚不符合自然语言规律的文本，保持语法和语义通顺
- 一致性 - 正确率关联: 模型对问题变种的多个回答中，一致性高的部分更可能是正确的

## 关联笔记
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: 同样涉及 DPO 训练方法，使用 SFT→DPO→GRPO 进行对齐
- 01KJBZRF4FT28F7NFSZD176TXQ.md: V-STaR 论文笔记，同样使用 DPO 训练验证器处理正确/错误答案对
- 01KJBZR1YB6S9NAWE04BZY5WY3.md: TPO 论文笔记，使用迭代式 DPO 训练模型的思考过程
