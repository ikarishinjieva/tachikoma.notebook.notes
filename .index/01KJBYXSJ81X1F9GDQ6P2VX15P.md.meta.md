---
note: 01KJBYXSJ81X1F9GDQ6P2VX15P.md
title: 20230409 - llama-index 逻辑整理
indexed_at: 2026-03-05T08:32:12.644271+00:00
---

## 标签
llama-index, RAG, 索引构建, 向量检索, embedding, 查询流程

## 摘要
详细梳理 llama-index 框架的核心逻辑，包括从文档创建索引（BaseGPTIndex.from_documents）到查询执行（BaseGPTIndex.query）的完整流程。涵盖 node 解析、embedding 生成、向量存储检索、响应合成等关键步骤。

## 关键概念
- BaseGPTIndex: 索引基类，提供 from_documents 和 query 核心接口
- Node Parser: 将文档切割成 text chunk 并转换为 node 的解析器
- Query Runner: 处理查询请求的执行器，负责 query 转换和路由到对应 index 的 query_obj
- Vector Store: 存储和检索 embedding 向量的组件，支持 top k 相似度搜索

## 关联笔记
- 01KJBZBMFN7QZYG0SXH1XM3S7Z.md: 同属 LlamaIndex 框架的 Property Graph Index 分析，涉及图谱索引类型
- 01KJBZEMNJW3ZAV821WQ5N70HB.md: 使用 llama_index.embeddings 模块，涉及 embedding 模型配置
- 01KJBZ68S0R9NS2HK7MWN06G5A.md: embedding 微调实践，与本文向量表示和检索优化相关
