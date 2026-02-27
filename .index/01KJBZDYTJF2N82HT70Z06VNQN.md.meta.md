---
note: 01KJBZDYTJF2N82HT70Z06VNQN.md
title: 20240811 - AutoPrompt架构
indexed_at: 2026-02-27T11:11:03.738765+00:00
---

用户要求我分析一篇关于 AutoPrompt 架构的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20240811 - AutoPrompt 架构
2. 笔记内容主要描述了 AutoPrompt 的优化步骤流程，包括：
   - OptimizationPipeline.run_pipeline
   - 调优步骤（annotator、predictor、eval、run_step_prompt 等）
   - 生成对抗样本的逻辑
   - 存在的问题和 TODO

3. 其他笔记列表都是关于 MySQL、Clickhouse、数据库、系统性能等方面的，与 AutoPrompt（提示词优化）没有直接关联。

现在我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：0-3 篇（看起来没有关联的）

让我生成这些内容。
## 标签
AutoPrompt, 提示词优化, LLM, 迭代优化, 对抗样本, 深度搜索

## 摘要
本文档记录了 AutoPrompt 的架构设计，包括优化流水线的执行流程和核心组件（标注器、预测器、评估器、提示词生成器）。同时指出了该算法采用深度搜索策略不会因评分下降而回退，以及样本充足时不触发对抗数据生成的问题。

## 关键概念
- OptimizationPipeline: 执行最多 num_steps 次调优步骤的优化流水线
- run_step_prompt: 根据错误分析生成新提示词的元提示模块
- 对抗样本生成: 当训练样本不足时生成挑战性样本以增强模型鲁棒性
- 深度搜索策略: 持续向下搜索最优提示词，不因中间评分下降而回退

## 关联笔记
无
