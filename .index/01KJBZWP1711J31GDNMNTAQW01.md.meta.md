---
note: 01KJBZWP1711J31GDNMNTAQW01.md
title: 20250807 - 阅读论文*: Beyond the 80/20 Rule: High-Entropy Minority Tokens Drive Effective Reinforcement Learning for LLM Rea
indexed_at: 2026-02-27T11:44:16.090229+00:00
---

用户要求我分析一篇关于 LLM 推理中熵模式的论文阅读笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主要内容**：论文分析 LLM 在 CoT 推理时的熵模式，发现高熵词元是推理的"分叉路口"，控制这些词元的熵可以提升推理性能。ms-swift 提供了相关参数控制。

2. **标签**：需要 3-6 个关键词，如：LLM、推理、熵、强化学习、思维链、token

3. **摘要**：2-3 句话概括核心内容

4. **关键概念**：3-5 个核心概念，如熵模式、高熵词元、低熵词元、分叉路口等

5. **关联笔记**：查看其他笔记列表，有一篇明确提到的相关论文笔记：`[20250807 - 阅读论文*: The Entropy Mechanism of Reinforcement Learning for Reasoning Language Models]`，但这份笔记列表中似乎没有这个文件。其他笔记都是关于 MySQL、Clickhouse、系统性能等主题，与这篇 LLM 论文笔记没有直接关联。

让我按要求格式输出。
## 标签
LLM, 推理, 熵, 强化学习, 思维链, Token

## 摘要
论文发现 LLM 推理时约 80% 词元为低熵（确定性高），仅 20% 为高熵（不确定性高）。高熵词元多为逻辑连接词，是推理路径的"分叉路口"，适度提高其随机性可提升推理性能。ms-swift 提供 `--top_entropy_quantile` 和 `--log_entropy` 参数支持该机制。

## 关键概念
- 熵模式: LLM 生成词元时的确定性分布规律
- 高熵词元: 不确定的逻辑连接词，决定推理方向的分叉点
- 低熵词元: 确定性的单词后缀、代码片段，起沿路执行作用
- 分叉路口: 高熵词元在推理路径中改变方向的关键位置

## 关联笔记
无
