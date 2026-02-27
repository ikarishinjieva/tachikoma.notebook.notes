---
note: 01KJBZR3H94N01JGGGAYA36GP2.md
title: 20250201 - 阅读论文: Chain-of-Thought Reasoning without Prompting
indexed_at: 2026-02-27T11:29:50.935644+00:00
---

用户希望我分析这篇关于 Chain-of-Thought Reasoning 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：关于 LLM 的 CoT（Chain-of-Thought）推理研究，特别是 CoT-Decoding 技术
2. **核心内容**：
   - 论文发现 LLM 词表中隐藏有 CoT 流程
   - 通过改变 decoder 来找到隐藏的 CoT
   - 与提示词中加入 CoT 的区别：CoT-Decoding 不依赖人工提示，更客观评估模型内在能力

3. **标签候选**：LLM、推理、CoT、解码策略、机器学习、论文阅读

4. **关键概念**：
   - CoT-Decoding：改变解码方式挖掘模型潜藏的推理能力
   - 贪婪算法：获取可能性最高的词
   - 自发 CoT：模型不依赖提示 tự产生的推理路径

5. **关联笔记**：查看其他笔记列表，主要是关于 MySQL、Clickhouse、系统性能等技术内容，与这篇 LLM/CoT 主题的笔记没有明显关联

让我生成简洁的索引元数据。## 标签
LLM, 推理, CoT, 解码策略, 论文阅读

## 摘要
论文提出 CoT-Decoding 技术，通过改变解码方式挖掘 LLM 词表中潜藏的自发推理路径。与提示词 CoT 不同，该方法无需人工干预即可更客观评估模型内在推理能力。

## 关键概念
- CoT-Decoding: 改变解码策略以挖掘模型潜藏的推理能力，无需额外训练或提示
- 贪婪算法: 传统解码方式，仅获取可能性最高的词，可能忽略隐藏的 CoT 路径
- 自发 CoT: 模型不依赖任何提示自主产生的推理流程

## 关联笔记
无
