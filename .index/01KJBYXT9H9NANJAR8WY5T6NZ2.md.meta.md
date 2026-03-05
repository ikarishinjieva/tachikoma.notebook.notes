---
note: 01KJBYXT9H9NANJAR8WY5T6NZ2.md
title: 20230406 - 测试 cohere + llama-index
indexed_at: 2026-03-05T08:33:42.137721+00:00
---

## 摘要
测试 Cohere LLM 和 embedding 模型与 llama-index 集成的效果，使用 MySQL 手册作为知识库进行索引和查询实验。分析了前处理阶段的 CPU 消耗（主要在 tokenize 阶段），记录构建索引耗时 1 小时以上、成本$2.17，但查询答案准确度不理想。

## 关键概念
- llama-index: 文档索引和查询框架，用于构建 RAG 系统
- Cohere: 提供 LLM 和 embedding 模型的 AI 服务提供商
- GPTSimpleVectorIndex: llama-index 的向量索引类型，用于文档检索
- ServiceContext: llama-index 中配置 LLM 和 embed_model 的统一入口
- LangchainEmbedding: 将 LangChain 的 embedding 模型适配到 llama-index 的包装器

## 关联笔记
- 01KJBYYMHN0C5CA4ARZ21B6AAN.md: 同系列测试笔记，使用 OpenAI/Azure 替代 Cohere 进行 llama-index 集成测试
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 后续问题排查，分析 llama-index 默认 response_mode 不使用 refine 的原因
- 01KJBYYZFEHS0QT21JG2ZJG2W9.md: 使用自定义 prompt 解决 llama-index 答案信息示踪问题
