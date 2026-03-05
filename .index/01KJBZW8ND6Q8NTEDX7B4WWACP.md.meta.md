---
note: 01KJBZW8ND6Q8NTEDX7B4WWACP.md
title: 20250801 - 进行SQL优化的GRPO - 缩小训练效果和校验效果的差异
indexed_at: 2026-03-05T11:48:46.236777+00:00
---

## 摘要
记录 SQL 优化 GRPO 训练的 4 次迭代尝试，包括基模型更换（Qwen3-8B-base、internlm3-8b-instruct、GLM-4-9B）、奖励函数设计（Action 约束、Action3F1、Completeness）和参数调优。核心问题是训练效果与评估效果存在显著差距，主要表现为生成长度限制、语句/列引用不一致。

## 关键概念
- GRPO: 强化学习训练方法，通过多奖励函数综合评估生成质量
- Action3F1: 奖励函数之一，匹配生成内容中动作 3 的语句和查询字段与 GT 的 F1 值
- Completeness: 奖励函数之一，评估必需动作（action1、action4）的存在性
- 生成长度限制: 评估数据 clip 比例高，导致 ActionCount 无法充分学习

## 关联笔记
- 01KJBZVXXN0557T4MAZ5HVYJMW.md: 前序笔记，对 GRPO 训练的 completion 进行分析，提出 warmup 阶段和训练/评估数据分布差异问题
