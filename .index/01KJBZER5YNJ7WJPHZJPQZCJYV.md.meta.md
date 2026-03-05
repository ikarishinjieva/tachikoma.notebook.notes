---
note: 01KJBZER5YNJ7WJPHZJPQZCJYV.md
title: 20240826 - 阅读论文: A Survey on In-context Learning
indexed_at: 2026-03-05T10:38:09.647275+00:00
---

## 摘要
该笔记记录了一篇关于 In-context Learning (ICL) 的综述论文，核心讨论提示词设计对 ICL 性能的影响。文章从示例组织（选择/重构/排序）、指令格式化、评分函数三个维度系统梳理了优化 ICL 的方法。

## 关键概念
- In-context Learning (ICL): 通过提示中的示例让模型学习任务，无需更新参数
- 示例组织: 通过选择、重构和排序示例来构建更有效的提示
- 思维链 (CoT): 引入中间推理步骤以增强模型推理能力的方法
- 通道模型 (Channel Models): 反向计算条件概率以估计答案可能性的评分方法

## 关联笔记
- 01KJBZYAS3THKM4CYDK96M7AQV.md: 同为 LLM 后训练论文阅读笔记，讨论了提示选择和响应优化策略
- 01KJC008VJ7N9T3VR0Q5C4YPK8.md: 同为 LLM 训练论文阅读笔记，研究了 RL 与 SFT 的遗忘机制差异
