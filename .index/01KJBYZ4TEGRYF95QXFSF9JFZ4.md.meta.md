---
note: 01KJBYZ4TEGRYF95QXFSF9JFZ4.md
title: 20230505 - 制作私有运维知识库的demo
indexed_at: 2026-02-27T10:28:44.529321+00:00
---

用户要求我分析这篇关于私有运维知识库 demo 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于制作私有运维知识库 demo 的笔记
2. 主要内容涉及：
   - 使用 llama_index 构建知识库
   - 与 chatbot 集成
   - 显示引用来源（行号追踪）
   - 使用 Azure OpenAI API
   - 使用 gradio 做 UI
   - 使用 mcontriever 做 embedding

3. 标签应该包括：私有知识库、llama_index、运维、RAG、引用追踪等

4. 摘要需要概括核心内容

5. 关键概念：llama_index、RAG、embedding、引用追踪、Azure OpenAI 等

6. 关联笔记：查看提供的笔记列表，主要是关于 MySQL、Clickhouse、数据库相关的笔记。这篇笔记是关于运维知识库的，与数据库运维相关，但没有直接的技术关联。可能可以关联一些数据库相关的笔记，因为这是运维知识库的内容来源。但根据内容，这篇笔记更多是关于如何构建知识库系统，而不是具体的数据库知识。所以关联笔记可能较少或无。

让我生成输出：## 标签
私有知识库, llama_index, RAG, 运维, 引用追踪, Azure OpenAI

## 摘要
记录使用 llama_index + Azure OpenAI 构建私有运维知识库 Demo 的过程，包括将索引嵌入 Chatbot、显示答案引用来源等技术实现。整理了待完成的功能清单（如图片处理、文档优先级、长上下文优化等）及核心代码片段。

## 关键概念
- llama_index: 用于构建大模型知识库索引的框架，支持向量检索和问答
- RAG (检索增强生成): 通过检索私有知识库增强大模型回答的准确性
- 引用追踪: 在原始文档中生成行号，让大模型标注答案来源的具体位置
- mcontriever: 用于多语言（中英文）的 embedding 模型
- DirectAgent: 自定义代理，将请求直接转发给 index 而非使用 langchain 现有 chain

## 关联笔记
无
