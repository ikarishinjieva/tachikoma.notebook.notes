---
title: 20240604 - 阅读论文: LLM-R: Learning to Retrieve In-Context Examples for Large Language Models
confluence_page_id: 2949590
created_at: 2024-06-04T07:07:15+00:00
updated_at: 2024-06-04T07:07:15+00:00
---

# 主要思路

  1. 目标: 提升 in-context learning时 (相当于提示词few-shot), 对召回文档的选取 (reranking)
  2. 主要思路1: 使用LLM对召回文档进行评分, 将评分能力蒸馏给 retriever模型
     1. 与其他目的的蒸馏相比, 以in-context learning为目标的蒸馏, 训练出的主要能力是: 是否应当选取该文档, 而不包括其他知识
  3. 主要思路2: 使用多轮迭代, 每一轮使用上一轮训练完成的模型
     1. 多轮迭代, 比起 单轮迭代多次训练, 多轮训练的优势是 训练数据是变化的, 不同轮次训练的能力侧面 可能不同
