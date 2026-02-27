---
note: 01KJBYYSWT0HJ1ZYSW82YFETXD.md
title: 20230429 - 文章解析: LLM的知识获取架构
indexed_at: 2026-02-27T10:27:48.896562+00:00
---

用户希望我分析这篇关于 LLM 知识获取架构的笔记，并生成索引元数据。让我仔细阅读笔记内容，然后按照指定格式输出。

笔记主要内容：
1. 讨论了 LLM 获取知识的两种主要方法（上下文和微调）及其缺点
2. 介绍了 llama_index 方案
3. 详细介绍了"假设文档嵌入"(Hypothetical Document Embeddings)方法
4. 解释了 Contriever、Dense Retriever 等概念
5. 涉及信息检索相关的技术

标签应该涵盖：LLM、知识检索、嵌入、llama_index、信息检索等

摘要需要概括核心内容

关键概念包括：假设文档嵌入、Dense Retriever、Contriever、llama_index 等

关联笔记：查看提供的笔记列表，主要是关于 MySQL、Clickhouse、系统性能等方面的笔记，与这篇 LLM 知识检索的笔记主题不太相关。可能没有直接关联的笔记。

让我按照格式输出。## 标签
LLM, 知识检索, 嵌入向量, llama_index, 信息检索, Dense Retriever

## 摘要
本文解析了 LLM 知识获取的架构方案，对比了上下文注入和微调两种主流方法的局限性。重点介绍了 llama_index 方案及假设文档嵌入（HyDE）等改进方法，并详细解释了 Dense Retriever 和 Contriever 等语义检索技术的原理。

## 关键概念
- 假设文档嵌入 (HyDE): 对问题生成虚假回答，然后在文档库中检索与虚假回答语义相似的真实文档
- Dense Retriever: 基于深度学习的信息检索方法，将文档和查询映射到高维向量空间进行语义匹配
- Contriever: Facebook 开发的上下文密集检索器，使用不同的分词器和编码器分别处理文档和查询
- llama_index: 一种 LLM 知识获取解决方案，提供多种索引结构来管理外部知识
- 词项匹配: 传统检索方法，基于词汇出现频率评估相似度，难以捕捉语义关系

## 关联笔记
无
