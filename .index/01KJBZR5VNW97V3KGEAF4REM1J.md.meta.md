---
note: 01KJBZR5VNW97V3KGEAF4REM1J.md
title: 20250202 - 阅读论文*: UNIPROMPT: Task Facet Learning: A Structured Approach to Prompt Optimization
indexed_at: 2026-03-05T11:21:08.202944+00:00
---

## 摘要
论文 UNIPROMPT 分析了提示优化问题的两个关键特性：LLM 对提示变化的敏感性（大模型更适合定向优化）和提示信息量与准确性的关系（章节形式添加任务方面符合边际效益递减）。基于此提出结构化提示词方法，用章节组织提示，通过迭代抽象错误 case 来生成或合并提示词章节。

## 关键概念
- 概率 Lipschitz 常数: 衡量 LLM 对提示变化敏感性的指标，值越低表示模型对提示微小变化越不敏感
- 定向优化算法: 利用性能反馈引导提示修改的优化方法，区别于随机搜索等非定向方法
- 结构化提示词: 用章节组织提示文档，每章节代表任务的某一方面，便于迭代优化
- 边际效益递减: 以章节形式添加任务方面时，LLM 性能提升随章节数量增加而逐渐减小的规律

## 关联笔记
- 01KJBZE0C1T7XST2M7PBZYEV9C.md: 同为提示优化论文（IPC），使用合成边界案例迭代优化提示词
- 01KJBZC8W6D9K78AHWW3G1S047.md: DSPy 框架的提示优化器实现（COPRO/MIPRO），与 UNIPROMPT 同属自动提示优化方向
- 01KJBZER5YNJ7WJPHZJPQZCJYV.md: 上下文学习综述，详细讨论提示设计对 ICL 性能的影响
