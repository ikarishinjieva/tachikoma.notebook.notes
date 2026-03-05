---
note: 01KJBZG2VSA818F7DC1JG7HF8X.md
title: 20241006 - 阅读论文*: EfficientRAG: Efficient Retriever for Multi-Hop Question Answering
indexed_at: 2026-03-05T10:45:31.166419+00:00
---

## 摘要
EfficientRAG 是一种针对多跳问答任务的高效检索框架，使用 Labeler/Tagger（词级二元分类器）和 Filter 两个轻量级组件替代 LLM 生成迭代查询。通过迭代检索 - 标记 - 过滤流程，在降低延迟和成本的同时保留最相关信息，最终由 LLM 生成答案。

## 关键概念
- Labeler: 词级别二元分类器，判断哪些词语是有用的信息
- Tagger: 二元分类器，判断文档块是否包含最终答案
- Filter: 生成下一跳查询并过滤无用信息的小模型
- 迭代检索: 多轮检索 - 生成循环，逐步获取解决问题所需的信息

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 包含迭代式 RAG 方法综述，介绍了 ITER-RETGEN 等同类迭代检索框架
- 01KJBZ5SXG26R34S13HPP05SYY.md: 详细描述 ITER-RETGEN 的迭代检索 - 生成协同机制，与 EfficientRAG 思路相似
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 同样针对多跳推理场景，但采用知识图谱+PPR 算法方案
