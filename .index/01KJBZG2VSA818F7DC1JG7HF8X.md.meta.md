---
note: 01KJBZG2VSA818F7DC1JG7HF8X.md
title: 20241006 - 阅读论文*: EfficientRAG: Efficient Retriever for Multi-Hop Question Answering
indexed_at: 2026-02-27T11:17:02.190486+00:00
---

用户要求我分析一篇关于 EfficientRAG 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：EfficientRAG 论文，关于多跳问答的高效检索器
2. **核心内容**：
   - 针对多跳问题的 RAG 改进，降低延迟/成本
   - 使用 Labeler + Tagger + Filter 替代 LLM 生成查询
   - 这些组件都是轻量级二元分类器

3. **标签**：RAG、多跳问答、信息检索、效率优化、论文阅读

4. **摘要**：需要概括 EfficientRAG 框架的核心思路和目标

5. **关键概念**：
   - EfficientRAG：高效检索框架
   - Labeler：词级别二元分类器，标记有用词语
   - Tagger：判断文档块是否包含答案
   - Filter：生成新查询并过滤无用信息
   - 迭代式检索

6. **关联笔记**：查看其他笔记列表，主要是：
   - 数据库相关（MySQL、Clickhouse）
   - 系统性能分析
   - 只有一篇 "TODO: Advanced Join Strategies for Large-Scale Distributed Computation" 可能与检索/查询相关，但关联度很低

大部分笔记都是数据库和系统相关的，与这篇 RAG 论文笔记关联度很低。## 标签
RAG, 多跳问答, 信息检索, 效率优化, 论文阅读, 轻量级模型

## 摘要
EfficientRAG 是一个针对多跳问答场景的高效检索框架，通过迭代生成查询并保留最相关信息来提高检索效率。其核心创新是使用轻量级的 Labeler、Tagger 和 Filter 组件替代 LLM 生成查询步骤，显著降低成本和延迟。

## 关键概念
- EfficientRAG: 包含 Labeler 和 Filter 两个轻量级组件的迭代式检索框架
- Labeler: 词级别二元分类器，标记哪些词语对回答问题有用
- Tagger: 判断文档块是否包含最终答案的二元分类器
- Filter: 基于标记结果生成下一跳查询并过滤无用信息
- 迭代式检索: 通过多轮检索 - 标记 - 过滤循环逐步收集答案所需信息

## 关联笔记
无
