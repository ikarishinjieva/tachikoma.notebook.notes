---
note: 01KJBZY39GJE38CB7BYSTRWMBX.md
title: 20250813 - 继续进行SQL优化的GRPO训练
indexed_at: 2026-02-27T11:44:59.924826+00:00
---

用户希望我分析这篇关于 SQL 优化 GRPO 训练的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是关于 SQL 优化的 GRPO（Group Relative Policy Optimization）训练笔记
2. 主要内容包括：
   - 前继工作：分析了训练时间/内存消耗问题
   - 需要减少 completion 长度来换取训练内存和时间
   - 修复尝试 11 中发现的训练/评估奖励崩溃问题
   - 尝试 12：减少模型生成长度为 6000，但出现了 eval F1 reward 为 0 的问题
   - 最后调整了参数配置

3. 关键概念：GRPO 训练、reward 函数、completion 长度、F1 reward、模型参数配置

4. 关联笔记：从提供的笔记列表中，这篇笔记主要是关于机器学习/GRPO 训练的，而其他笔记大多是关于 MySQL、Clickhouse、系统性能等数据库相关内容，没有直接关联的笔记。

让我生成符合要求的索引元数据。
## 标签
GRPO 训练，SQL 优化，奖励函数，内存优化，模型参数

## 摘要
记录了 GRPO 训练中内存/时间优化及奖励函数修复的进展。通过调整 max_length 等参数解决 eval reward 异常问题，并完善了 reward 函数的多列匹配和删除判断逻辑。

## 关键概念
- GRPO 训练：基于组相对策略优化的强化学习方法，用于 SQL 优化任务
- Reward 函数：评估模型输出的奖励机制，支持多列模式和多种删除表述匹配
- Completion 长度：模型生成的最大 token 数，影响训练内存和时间消耗
- F1 Reward：评估生成结果与标准答案匹配度的指标
- 参数配置：max_length/max_completion_length/vllm_max_model_len 的组合调整

## 关联笔记
无
