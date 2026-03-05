---
note: 01KJBZ5XTNYJSRXZ72WTVJ391G.md
title: 20240210 - 阅读论文: STEP-BACKPROMPTING
indexed_at: 2026-03-05T09:33:33.277554+00:00
---

## 摘要
论文提出 STEP-BACK PROMPTING 方法，通过让 LLM 先对问题进行抽象、获取高级概念或原则，再基于这些抽象信息进行推理，从而简化复杂多步推理任务。实验证明该方法在 MMLU 物理/化学、TimeQA、MuSiQue、StrategyQA 等任务上显著优于基线和 GPT-4。

## 关键概念
- STEP-BACK PROMPTING: 通过将具体问题抽象为高层次问题，获取通用原理后再推理的 prompting 方法
- 抽象步骤: LLM 对原始问题进行抽象，生成涵盖高级概念的通用问题并回答
- 推理步骤: 利用抽象步骤获得的高级概念或原则指导原始问题的推理
- 多跳推理: 需要跨越多个概念和领域、经过多个推理步骤才能解答的复杂任务

## 关联笔记
- 01KJBZA9VK4YZ0YE2EXTHFM3X1.md: 实际应用中尝试用 step-back 方法重写数据库问题以改进检索
- 01KJBZ65Y2QFCFWKWQVM7T6NQ3.md: 拆解问题要素方法与 STEP-BACK 类似，都将问题抽象化以提高检索命中率
