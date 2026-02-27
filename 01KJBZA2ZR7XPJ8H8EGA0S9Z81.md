---
title: 20240614 - 阅读论文*: Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity
confluence_page_id: 2949700
created_at: 2024-06-14T06:18:59+00:00
updated_at: 2024-06-14T06:18:59+00:00
---

主要思路: 

通过对问题复杂度的分类器, 使用不同的策略: 

  - 无检索：直接使用 LLM 生成答案，适用于最简单的问题。
  - 单步检索：检索与问题相关的文档，并将其输入 LLM 生成答案，适用于中等复杂度的问题。
  - 多步检索：迭代地检索文档并生成中间答案，直到最终答案生成，适用于最复杂的问题。

分类器的训练数据集, 可以用样例问题 经过不同策略的答案 的对比效果, 决定什么策略对训练数据是最合适的. (也就是说训练数据不需要进行人工标记, 可以进行自动标记)
