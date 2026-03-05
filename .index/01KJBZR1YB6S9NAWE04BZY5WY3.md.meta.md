---
note: 01KJBZR1YB6S9NAWE04BZY5WY3.md
title: 20250131 - 阅读论文: TPO: THINKING LLMS: GENERAL INSTRUCTION FOLLOWING WITH THOUGHT GENERATION
indexed_at: 2026-03-05T11:18:49.897761+00:00
---

## 摘要
TPO 论文提出在 DPO 框架中显式引入"思考过程"训练，通过评判模型对回复评分间接优化思考质量，无需标注的思考数据。采用迭代式训练方式，使模型逐步学会利用思考提升回复质量。

## 关键概念
- TPO: 将思考过程显式纳入 DPO 训练框架的方法
- 迭代式 DPO: 多轮迭代训练，逐步优化思考过程和回复质量
- 间接优化: 利用回复评分间接评估思考过程质量，无需标注数据
- 思考 - 回复结构: 模型输出包含思考过程和最终回复两部分

## 关联笔记
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: 同样涉及 DPO→GRPO 的训练流程和对齐方法
- 01KJBZZJ92R5GJ0VH1J4W3X93K.md: 使用 GRPO 训练生成思考过程，与 TPO 思路相似
