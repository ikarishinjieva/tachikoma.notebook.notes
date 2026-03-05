---
note: 01KJBZJG5YZMGQK9DSR57ZV9ZR.md
title: 20241107 - 阅读论文: Training Verifiers to Solve Math Word Problems
indexed_at: 2026-03-05T10:55:45.589255+00:00
---

## 摘要
该论文提出设计一个评分器（验证器）来评估数学应用题的解答过程和结果。核心思路是通过训练 verifier 模型来判断解题答案的正确性，从而提升模型的数学推理能力。

## 关键概念
- 评分器/验证器: 用于评估数学问题解答过程和结果正确性的模型
- 数学应用题: 需要多步推理才能解决的数学文字问题
- 解答评估: 对解题思路和最终答案进行准确性判断

## 关联笔记
- 01KJBZRF4FT28F7NFSZD176TXQ.md: 同主题论文笔记，研究用 DPO 训练验证器来评估推理答案
- 01KJBZRFKPHCGB3ETQ4MS9CVH0.md: 相关论文笔记，使用 Verifier 评估数学题解题思路质量
