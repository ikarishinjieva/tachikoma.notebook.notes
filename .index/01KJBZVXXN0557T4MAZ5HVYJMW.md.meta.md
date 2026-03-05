---
note: 01KJBZVXXN0557T4MAZ5HVYJMW.md
title: 20250727 - 进行SQL优化的GRPO - 对completion进行分析
indexed_at: 2026-03-05T11:47:35.668757+00:00
---

## 摘要
分析 GRPO 训练尝试 12 的 completion 结果，发现训练仅使用 134/187 条数据（怀疑 dynamic_sample 问题）。从低分 completion 中识别出 5 类问题：SQL 含 DDL 语句、GT 表名引用层次错误、GT 语义不合理、多 SQL case 处理、GT 格式非法。

## 关键概念
- GRPO: 组相对策略优化，通过组内响应比较计算优势，无需独立价值模型
- completion 分析: 检查模型生成的训练输出，用于诊断训练问题和数据质量
- dynamic_sample: 动态采样机制，可能导致部分训练数据未被使用
- 奖励函数: 用于评估 SQL 优化质量的评分函数，影响 GRPO 训练方向
- GT(Ground Truth): 标准答案，用于计算奖励和验证模型输出正确性

## 关联笔记
- 01KJBZV98B1VCTQQ6ZAB998E7T.md: 前序训练尝试 (尝试 12)，本次分析的直接对象
- 01KJBZW8ND6Q8NTEDX7B4WWACP.md: 基于本次分析结果，继续优化训练数据和奖励函数
