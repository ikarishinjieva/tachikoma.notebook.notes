---
note: 01KJBZVSMEPVZM9JAKWFXTSJMD.md
title: 20250724 - 对GRPO训练中grad_norm的理解
indexed_at: 2026-03-05T11:47:13.722401+00:00
---

## 摘要
分析 GRPO 训练中 grad_norm 曲线的三个阶段：初始快速下降、低位平稳震荡、周期性梯度尖峰。低梯度范数基线表明 KL 正则化有效且模型进入精调阶段，周期性尖峰证明模型仍有学习潜力，支持续训决策。

## 关键概念
- 梯度范数 (grad_norm): 衡量梯度大小的指标，初始值 0.19 后降至 0.04-0.1 区间
- KL 惩罚: GRPO 中防止模型偏离参考模型太远的正则化项，会压低梯度范数
- 梯度尖峰: 周期性出现的梯度冲高现象，对应高价值数据批次和强烈学习信号
- 梯度裁剪: 防止梯度爆炸的护栏机制，常见阈值为 1.0 或 10.0

## 关联笔记
- 01KJBZVR7G5C69KG304370BJ9D.md: GRPO 基础概念和训练核心步骤的详解
- 01KJBZW65ZX6JQ36Y0ZFCEENMB.md: GRPO 中 completion 文件和 step/epoch 计算逻辑
- 01KJBZY39GJE38CB7BYSTRWMBX.md: SQL 优化的 GRPO 训练实战，包含 grad_norm 日志数据
