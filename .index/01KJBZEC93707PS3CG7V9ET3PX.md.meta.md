---
note: 01KJBZEC93707PS3CG7V9ET3PX.md
title: 20240815 - 阅读论文*: LARGE LANGUAGE MODELS ARE HUMAN-LEVEL PROMPT ENGINEERS
indexed_at: 2026-02-27T11:12:03.672346+00:00
---

用户要求我分析一篇关于 LLM 提示词工程论文阅读的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：关于 APE (Automatic Prompt Engineer) 论文的阅读笔记
2. 核心内容：
   - APE 生成候选提示词的两种模式：正向生成和反向生成
   - 迭代优化过程中的候选指令生成方法
   - 对算法设计的讨论（为什么用两种生成模式、如何避免局部最优）
   - TODO: 尝试运行项目

3. 标签应该是：LLM, 提示词工程, APE, 论文阅读, 自动化

4. 摘要：概括 APE 论文的核心思路，包括候选提示词生成和 beam search 优化

5. 关键概念：
   - 正向生成
   - 反向生成
   - 语义相似度重采样
   - Beam search

6. 关联笔记：查看提供的笔记列表，大部分是关于 MySQL、Clickhouse、数据库相关的，与这篇 LLM/提示词工程的笔记没有明显关联。有一篇 "20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation" 也是 TODO 类型的，但主题不相关。所以关联笔记应该是「无」。

让我按照格式输出。## 标签
LLM, 提示词工程, APE, 论文阅读, 自动化, Beam Search

## 摘要
介绍 APE (Automatic Prompt Engineer) 论文的核心思路：通过正向/反向生成模式产生候选提示词，并用 beam search 找到最优解。笔记记录了候选指令生成的两阶段方法及对算法设计的深入讨论。

## 关键概念
- 正向生成 (Forward Generation): 引导 LLM 从少量样本从头开始生成指令
- 反向生成 (Reverse Generation): 在 prompt 中预留空位，引导 LLM 填补指令
- 语义相似度重采样: 基于已有指令生成语义相似的变体用于迭代优化
- Beam Search: 搜索算法，用于从候选提示词中找到最优解

## 关联笔记
无
