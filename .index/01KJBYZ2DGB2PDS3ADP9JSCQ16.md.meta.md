---
note: 01KJBYZ2DGB2PDS3ADP9JSCQ16.md
title: 20230503 - 测试方法"Hypothetical Document Embeddings" (HyDE)
indexed_at: 2026-03-05T08:41:07.149727+00:00
---

## 标签
HyDE, Hypothetical Document Embeddings, LangChain, 检索增强，嵌入生成，LLM 测试

## 摘要
记录 HyDE (Hypothetical Document Embeddings) 方法的测试过程，包括在 LangChain 中的实现代码和 OpenAI API 调用日志。通过生成假设性文档片段来改进查询嵌入，对比了启用/禁用 HyDE 后的检索效果差异。

## 关键概念
- HyDE (Hypothetical Document Embeddings): 用 LLM 生成假设性答案文档，再对假设文档进行嵌入以改进检索
- HypotheticalDocumentEmbedder: LangChain 中实现 HyDE 方法的嵌入器类，劫持查询的 embedding 流程
- mContriever: 用于多语言检索任务的密集检索器，与 HyDE 配合使用
- 提示词工程: 使用"Please write a passage to answer the question"模板引导 LLM 生成假设文档

## 关联笔记
- 01KJBYYSWT0HJ1ZYSW82YFETXD.md: 早期文章解析笔记，介绍 HyDE 方法原理和 Contriever 检索器
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 包含相同的 HyDE 测试代码实现，使用 LangChain 和 llama_index
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 输入增强方法汇总，将 HyDE 归类为查询改写技术之一
