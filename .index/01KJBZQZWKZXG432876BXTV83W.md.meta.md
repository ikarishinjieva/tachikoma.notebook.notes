---
note: 01KJBZQZWKZXG432876BXTV83W.md
title: 20250131 - 阅读论文*: ZEUS: Enhancing Zero-shot Chain of Thought Prompting via Uncertainty-Guided Strategy Selection
indexed_at: 2026-02-27T11:28:46.731033+00:00
---

用户要求我分析这篇关于 ZEUS 论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 ZEUS 论文的笔记，ZEUS 是一个用于增强零样本思维链（Zero-shot Chain of Thought）prompting 的方法
2. 核心思想是通过不确定性估计来选择高质量的 few-shot demonstrations
3. 主要流程包括：不确定性估计（多重扰动）、样本选择、构建 demonstrations、推理

现在我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

看其他笔记列表，都是关于数据库、系统调试、Clickhouse 等技术内容，与这篇 AI/LLM 论文笔记没有明显关联。

让我生成输出：

标签：大语言模型，思维链，不确定性估计，Few-shot Learning，Prompt 工程

摘要：ZEUS 通过多重扰动（温度、提示词、问题改写）估计问题不确定性，并基于不确定性筛选高质量样本构建 few-shot demonstrations。该方法无需额外训练，仅通过样本选择策略即可提升零样本 CoT prompting 的推理性能。

关键概念：
- 不确定性估计：使用预测熵衡量模型对问题答案的不确定程度


- 多重扰动：通过温度、提示词、问题改写三种方式生成多个答案
- 样本选择策略：基于不确定性的均值和标准差选择特定范围的问题

关联笔记：无（其他笔记都是数据库和系统相关内容，与这篇 AI 论文无关）## 标签
大语言模型，思维链，不确定性估计，Few-shot Learning，Prompt 工程

## 摘要
ZEUS 通过多重扰动（温度、提示词、问题改写）估计问题不确定性，并基于不确定性筛选高质量样本构建 few-shot demonstrations。该方法无需额外训练，仅通过样本选择策略即可提升零样本 CoT prompting 的推理性能。

## 关键概念
- 不确定性估计：使用预测熵衡量模型对问题答案的不确定程度，越高表示越不确定
- 多重扰动：通过温度、提示词、问题改写三种方式生成多个答案以评估不确定性
- 样本选择策略：基于不确定性的均值和标准差选择特定范围的问题作为 demonstration

## 关联笔记
无
