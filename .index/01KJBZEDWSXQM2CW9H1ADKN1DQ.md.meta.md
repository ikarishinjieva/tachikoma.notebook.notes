---
note: 01KJBZEDWSXQM2CW9H1ADKN1DQ.md
title: 20240819 - 阅读论文: Principled Instructions Are All You Need for Questioning LLaMA-1/2, GPT-3.5/4
indexed_at: 2026-03-05T10:34:39.214872+00:00
---

## 标签
提示词工程，设计原则，大语言模型，LLM 指令优化，Prompt Engineering

## 摘要
该笔记记录了论文《Principled Instructions Are All You Need for Questioning LLaMA-1/2, GPT-3.5/4》中提出的 26 条提示词设计原则，归纳为五大类：结构与清晰度、具体性和信息量、用户交互、内容风格、复杂任务与代码。笔记同时分析了论文的 related works，涵盖 AutoPrompt、Few-shot Learning、Least-to-Most Prompting 等关键研究方向。

## 关键概念
- 肯定指令 (Affirmative Directives): 使用"做..."等肯定指令，避免否定语言
- 思维链 (Chain-of-Thought): 将思维链与少样本提示结合，引导逻辑推理
- 增量提示 (Incremental Prompting): 将复杂任务分解为简单步骤逐步引导
- 输出引导 (Output Primers): 在提示词末尾加入预期输出开头引导生成
- 任务分解 (Task Decomposition): 将复杂任务拆分为多个子任务逐步完成

## 关联笔记
- 01KJBZ63853E2DE7AXYHDQ1PGM.md: Chain-of-Thought Prompting 论文笔记，思维链是本笔记 26 条原则之一
- 01KJBZEFVYBDN8ZJD2E8A25KB3.md: Prompt Pattern Catalog 论文笔记，在本笔记 related works 中被引用
- 01KJBZEEWEA30G6D7KYV9NNZ35.md: Least-to-Most Prompting 论文笔记，在本笔记 related works 中被引用
