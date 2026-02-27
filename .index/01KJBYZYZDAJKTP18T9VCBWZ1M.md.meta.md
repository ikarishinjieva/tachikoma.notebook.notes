---
note: 01KJBYZYZDAJKTP18T9VCBWZ1M.md
title: 20230514 - 制作私有运维知识库的demo (2)
indexed_at: 2026-02-27T10:29:05.281554+00:00
---

我需要分析这篇笔记，提取索引元数据。让我仔细阅读内容：

这是一篇关于"私有运维知识库 demo"的笔记，主要内容包括：
1. 人工修订文档的进展和问题
2. TODO 列表（3 个未完成的任务）
3. 文档预处理计划（summary、重写）
4. 获取摘要的代码和效果分析
5. 文档重写的尝试
6. streaming 功能的技术问题（堆栈跟踪）

核心主题：运维知识库、文档处理、LLM/RAG 系统

标签应该包括：运维知识库、文档处理、LLM、RAG、streaming 等

关键概念：
- llama_index: 用于构建 RAG 系统的框架
- map-reduce: 文档切片处理方法
- streaming: 实时输出功能
- token: 大模型上下文长度单位
- confluence storage format: 文档存储格式

关联笔记：从提供的笔记列表看，这篇笔记主要是关于知识库构建和 LLM 应用，与列表中的 MySQL、Clickhouse、性能分析等笔记关联度不高。可能需要找一些更相关的，但列表中没有特别匹配的。

让我生成输出：
## 标签
运维知识库，文档处理，LLM, RAG, streaming, llama_index

## 摘要
记录私有运维知识库 demo 开发进展，包括人工修订文档测试效果及发现的问题。分析了文档预处理（摘要、重写）的技术难点，并追踪了 streaming 功能无法触发的堆栈问题。

## 关键概念
- llama_index: 用于构建检索增强生成 (RAG) 系统的框架
- map-reduce: 文档切片后合并处理的策略，存在章节层次丢失问题
- streaming: 实时流式输出功能，在 tool 调用链中无法触发监听
- confluence storage format: Confluence 的 HTML 存储格式，需特殊解析处理
- token 超长: 上下文长度超出限制导致报错，与聊天历史长度相关

## 关联笔记
无
