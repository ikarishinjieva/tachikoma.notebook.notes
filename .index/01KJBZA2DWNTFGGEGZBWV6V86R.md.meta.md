---
note: 01KJBZA2DWNTFGGEGZBWV6V86R.md
title: 20240614 - 阅读论文: Self-Knowledge Guided Retrieval Augmentation for Large Language Models
indexed_at: 2026-03-05T10:09:35.630526+00:00
---

## 摘要
提出通过划定 LLM 知识边界来判断何时需要外部检索的框架。核心方法是对比有/无 RAG 时的答案效果，确定哪些问题属于 LLM 自有知识。提供四种知识边界建模方法：直接提问、上下文学习、训练分类器、最近邻搜索。

## 关键概念
- 知识边界: LLM 自有知识与需外部检索知识的分界线
- 直接提问: 直接询问 LLM 是否需要额外信息来回答问题
- 上下文学习: 通过示例引导 LLM 判断是否需要额外信息
- 训练分类器: 使用已知问题和标签训练分类器预测检索需求
- 最近邻搜索: 通过语义相似度匹配已知问题来判断检索需求

## 关联笔记
- 01KJBZWJBAGGD1TA8SGM6YZXR4.md: KAG-Thinker 也采用知识边界判定机制，使用"先生成后评估"策略决定是否需要外部检索
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 的微调+RAG 方案同样涉及何时需要检索的决策问题
