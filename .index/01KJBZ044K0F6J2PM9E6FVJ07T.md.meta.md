---
note: 01KJBZ044K0F6J2PM9E6FVJ07T.md
title: 20230523 - llama_index为什么不使用refine
indexed_at: 2026-02-27T10:29:22.565039+00:00
---

用户要求我分析一篇关于 llama_index 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：llama_index 为什么不使用 refine
2. 核心内容：
   - llama_index 的 response_mode 默认值是 compact，会合并多个文章片段
   - 当片段较少时被压缩到一个 query 中，不会使用 refine
   - 调大 similarity_top_k 会返回更多片段，会用到 refine
   - 但调大 similarity_top_k 会导致超过 max_token 错误

3. 标签应该包括：llama_index, RAG, refine, query, token 等

4. 关联笔记：看其他笔记列表，主要是 MySQL、Clickhouse、数据库相关的，没有与 llama_index 或 LLM 相关的笔记，所以关联笔记应该是「无」

让我生成输出：## 标签
llama_index, RAG, refine, query_engine, token 限制, similarity_top_k

## 摘要
分析了 llama_index 未使用 refine 模式的原因：默认 response_mode 为 compact，在片段较少时会合并到一个 query 中。调大 similarity_top_k 可触发 refine，但会导致 LLM 调用超过 max_token 限制。

## 关键概念
- response_mode: 控制 llama_index 如何合并多个片段的响应生成策略
- compact: 默认模式，将多个文章片段合并压缩后一次性处理
- refine: 逐片迭代优化响应的模式，适合处理大量片段
- similarity_top_k: 控制检索返回的相似片段数量

## 关联笔记
无
