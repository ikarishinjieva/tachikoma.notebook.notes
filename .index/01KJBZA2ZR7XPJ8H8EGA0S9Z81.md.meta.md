---
note: 01KJBZA2ZR7XPJ8H8EGA0S9Z81.md
title: 20240614 - 阅读论文*: Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity
indexed_at: 2026-03-05T10:09:57.502230+00:00
---

## 摘要
Adaptive-RAG 通过问题复杂度分类器动态选择检索策略：简单问题直接回答、中等问题单步检索、复杂问题多步迭代检索。分类器使用自动标注的训练数据学习，无需人工标记即可为不同问题匹配最优策略。

## 关键概念
- 问题复杂度分类器: 根据问题难度自动选择检索策略的核心组件
- 无检索策略: 直接调用 LLM 生成答案，适用于简单问题
- 单步检索: 单次检索相关文档后输入 LLM，适用于中等复杂度问题
- 多步检索: 迭代检索并生成中间答案直至最终输出，适用于复杂问题
- 自动标注训练: 通过对比不同策略在样例问题上的效果自动标注训练数据

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记，在 RAG 流程增强部分提及 Adaptive-RAG 作为自适应策略的代表方法
- 01KJBZA2394KZG8T9YYWXT1CXS.md: 同日期阅读的另一篇自适应检索论文，关注用自适应检索缓解 LLM 幻觉问题
- 01KJBZ5717N16C7ZP3KFQKF32G.md: 讨论迭代检索和多步检索技术，与 Adaptive-RAG 的多步检索策略相关
