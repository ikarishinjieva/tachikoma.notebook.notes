---
note: 01KJBZR3H94N01JGGGAYA36GP2.md
title: 20250201 - 阅读论文: Chain-of-Thought Reasoning without Prompting
indexed_at: 2026-03-05T11:19:59.867350+00:00
---

## 摘要
论文发现 LLM 贪婪解码会隐藏模型自发的 CoT 推理路径，提出通过改变解码方式（CoT-Decoding）来挖掘这些隐藏的推理链。该方法不依赖提示词工程，能更客观评估模型内在推理能力。

## 关键概念
- CoT-Decoding: 通过改变解码策略而非提示词来激发模型自发的思维链推理
- 贪婪解码: 传统解码方式只选择概率最高的 token，可能遗漏隐藏的 CoT 路径
- 自发 CoT: 模型不依赖人工提示词、自身产生的推理链条
- Top-k 解码: 探索多个高概率初始响应以发现隐藏的推理路径

## 关联笔记
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 同属模型推理机制研究，该笔记探讨用归因图追踪 LLM 内部思考过程，与本笔记的 CoT-Decoding 目标一致
