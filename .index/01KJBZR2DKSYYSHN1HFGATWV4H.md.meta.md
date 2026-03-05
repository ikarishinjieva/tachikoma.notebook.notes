---
note: 01KJBZR2DKSYYSHN1HFGATWV4H.md
title: 20250201 - 阅读论文*: STaR: Self-Taught Reasoner Bootstrapping Reasoning With Reasoning
indexed_at: 2026-03-05T11:19:22.680952+00:00
---

## 摘要
STaR 方法通过让模型生成推理过程，筛选正确答案作为训练数据进行迭代训练，逐步提升推理能力。该方法模拟教师引导学生学习的过程，循环执行生成推理、筛选正确案例、重新训练三个阶段。

## 关键概念
- STaR (Self-Taught Reasoner): 一种通过迭代训练让模型自学推理的方法
- 迭代训练: 多轮循环生成推理数据并筛选正确答案进行训练
- 推理化: 对错误题目提供正确答案让模型补充推理过程
- 筛选机制: 只保留推理正确且答案正确的案例作为训练数据

## 关联笔记
- 01KJBZRF4FT28F7NFSZD176TXQ.md: V-STaR 论文笔记，在 STaR 基础上引入验证器利用错误答案进行 DPO 训练
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 思维链追踪论文，研究模型内部推理机制与 CoT 忠实性验证
