---
note: 01KJBZEEWEA30G6D7KYV9NNZ35.md
title: 20240819 - 阅读论文: LEAST-TO-MOST PROMPTING ENABLES COMPLEX REASONING IN LARGE LANGUAGE MODELS
indexed_at: 2026-03-05T10:35:18.568815+00:00
---

## 标签
Least-to-Most Prompting, 思维链 (CoT), 复杂推理，问题分解，大语言模型，泛化能力

## 摘要
论文指出 CoT 在解决比示例更难的问题时性能显著下降，因为模型无法有效泛化超出示例复杂度的推理模式。提出 Least-to-Most Prompting 方法，将复杂问题拆解为多个子问题逐一求解，从而提升模型的复杂推理能力。

## 关键概念
- Least-to-Most Prompting: 将复杂问题分解为子问题，依次求解每个子问题的提示方法
- 思维链 (CoT): 通过让模型展示推理步骤来提升推理能力的提示技术
- 泛化瓶颈: CoT 在测试集复杂度超过提示示例时性能急剧下降的现象
- 问题分解: 将复杂任务拆解为可独立求解的子任务的策略

## 关联笔记
- 01KJBZETD71Z66AHRQNQ181WNG.md: 同样研究 CoT prompting，提出自动构建 CoT 样例的方法
- 01KJBZS20MPESNN97X6XPEZBV0.md: 讨论 CoD (Chain of Draft)，是 CoT 的简洁变体，优化思考效率
