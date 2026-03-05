---
note: 01KJBZV98B1VCTQQ6ZAB998E7T.md
title: 20250715 - 重新进行SQL优化的GRPO
indexed_at: 2026-03-05T11:46:35.754279+00:00
---

## 摘要
记录 SQL 优化任务中 GRPO 强化学习的多轮训练尝试，从尝试 1 到尝试 5，探索学习率、epoch、generation 数等参数配置。发现 OOM 问题、reward 解析错误、验证数据生成缺陷等问题，逐步修正训练数据生成流程和奖励函数逻辑。

## 关键概念
- GRPO: 基于群组的强化学习优化算法，用于 SQL 改写任务的多步推理训练
- 奖励函数 (Reward Function): 根据模型输出的变更决定与 GT 变更列表比对进行评分
- Thinking 步骤: 定义模型对 SQL 结构的多维度分析流程（计算列分析、列引用分析、优化变更等）
- JSON 结构标注: 将 SQL 转换为带编号的 JSON 格式，增强 LLM 对 SQL 层次结构的理解
- OOM (Out of Memory): 训练过程中因 generation 数和 batch size 过大导致的显存溢出问题

## 关联笔记
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 前继笔记，继续进行 SQL 优化的 GRPO 训练，修复尝试 11 中的奖励崩溃问题
- 01KJBZZJ92R5GJ0VH1J4W3X93K.md: 后续笔记，重新制作数据生成过程，根据输入输出 SQL 生成思考过程
- 01KJC00DJ026W6BGSW0TEW00GJ.md: 后续笔记，生成具有挑战的 case，平衡 SQL 各种劣化方向的处理方法
