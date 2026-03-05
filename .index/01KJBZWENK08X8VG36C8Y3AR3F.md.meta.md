---
note: 01KJBZWENK08X8VG36C8Y3AR3F.md
title: 20250804 - 对GSPO的理解
indexed_at: 2026-03-05T11:49:00.981686+00:00
---

## 标签
GRPO, GSPO, 强化学习, 重要性采样, 序列级优化, PPO

## 摘要
分析 GRPO 算法的核心机制：通过组内相对优势计算避免价值模型需求，但指出其 token 级重要性权重设计存在高方差噪声累积问题。GSPO 提出将 token 级修正改为序列级差异以增强训练效果。

## 关键概念
- GRPO: 组相对策略优化，通过同一查询的多个响应计算组内相对优势，无需价值模型
- 重要性采样: 用旧策略生成样本修正新策略期望估计，需要多样本才能可靠估计


- Token 级权重: GRPO 对每个 token 单独计算重要性权重，导致高方差噪声累积
- GSPO: 将 token 级修正改为序列级差异，解决 GRPO 的权重设计问题
- 价值模型: PPO 中用于评估状态/动作优势的模型，GRPO 通过组内比较避免其需求

## 关联笔记
- 01KJBZWQZ9BA47ET5VQBKGXEVE.md: 笔记中明确引用的 ms-swift GRPO compute_loss 函数分析
- 01KJBZVR7G5C69KG304370BJ9D.md: GRPO 基础理解笔记，提供算法背景知识
- 01KJBZY39GJE38CB7BYSTRWMBX.md: SQL 优化场景下 GSPO 参数的实际应用探索## 标签
GRPO, GSPO, 强化学习, 重要性采样, 序列级优化, PPO

## 摘要
分析 GRPO 算法的核心机制：通过组内相对优势计算避免价值模型需求，但指出其 token 级重要性权重设计存在高方差噪声累积问题。GSPO 提出将 token 级修正改为序列级差异以增强训练效果。

## 关键概念
- GRPO: 组相对策略优化，通过同一查询的多个响应计算组内相对优势，无需价值模型
- 重要性采样: 用旧策略生成样本修正新策略期望估计，需要多样本才能可靠估计
- Token 级权重: GRPO 对每个 token 单独计算重要性权重，导致高方差噪声累积
- GSPO: 将 token 级修正改为序列级差异，解决 GRPO 的权重设计问题
- 价值模型: PPO 中用于评估状态/动作优势的模型，GRPO 通过组内比较避免其需求

## 关联笔记
- 01KJBZWQZ9BA47ET5VQBKGXEVE.md: 笔记中明确引用的 ms-swift GRPO compute_loss 函数分析
- 01KJBZVR7G5C69KG304370BJ9D.md: GRPO 基础理解笔记，涉及信用分配和对称裁剪机制
- 01KJBZY39GJE38CB7BYSTRWMBX.md: SQL 优化场景中 GSPO 参数调整的实际训练探索
