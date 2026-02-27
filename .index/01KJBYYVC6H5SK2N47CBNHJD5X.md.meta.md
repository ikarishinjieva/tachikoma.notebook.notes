---
note: 01KJBYYVC6H5SK2N47CBNHJD5X.md
title: 20230501 - 测试 mcontriever + openai + llama_index
indexed_at: 2026-02-27T10:27:57.454409+00:00
---

用户希望我分析这篇笔记并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于测试 llama_index + openai + embedding 模型的技术笔记
2. 主要测试了 HuggingFaceInstructEmbeddings 和 HuggingFaceEmbeddings 两种 embedding 模型
3. 使用中文问题从英文语料（MySQL 官方文档）中获取信息
4. 发现了 query 接口 top_k 参数默认值为 1 的问题
5. 发现了模型可能生成错误信息的问题
6. 列出了下一步要解决的问题

标签应该包括：llama_index、embedding、MySQL、RAG、检索增强生成等

摘要需要概括核心内容

关键概念包括：HuggingFaceEmbeddings、similarity_top_k、Node、query 接口等

关联笔记：从提供的笔记列表看，这些都是 2021 年的笔记，而这篇是 2023 年 5 月的。内容上，这篇笔记主要关于 llama_index 和 embedding 测试，与 MySQL 相关。从列表中看，有几篇 MySQL 相关的笔记可能有关联：
- 20210404 - MySQL 刷脏页研究
- 20210330 - performance_schema 的日常使用
- 20210329 - MySQL 刷盘研究

但这些笔记主要是 MySQL 内部机制研究，而当前笔记是关于使用 RAG 技术查询 MySQL 文档的，关联度不高。

不过笔记内容涉及 MySQL 官方文档的检索，所以可以选取几篇 MySQL 相关的作为关联。
## 标签
llama_index, embedding, MySQL, RAG, 检索增强生成，大模型

## 摘要
测试使用 llama_index 配合不同 HuggingFace embedding 模型，从 MySQL 英文官方文档中进行中文问答检索。发现 query 接口 similarity_top_k 参数默认值为 1 导致上下文不足，且模型可能生成文档中不存在的信息。

## 关键概念
- HuggingFaceEmbeddings: 用于文本向量化表示的嵌入模型
- similarity_top_k: 控制检索返回的相似节点数量
- Node: llama_index 中检索和生成上下文的基本单元
- query 接口: 基于检索的问答接口，区别于 chat 接口

## 关联笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同为 MySQL 刷脏页相关研究，涉及 MySQL 内部机制
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同为 MySQL performance_schema 相关研究
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 同为 MySQL 相关技术学习记录
