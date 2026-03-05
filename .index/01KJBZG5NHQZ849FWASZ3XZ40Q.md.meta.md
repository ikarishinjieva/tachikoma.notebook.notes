---
note: 01KJBZG5NHQZ849FWASZ3XZ40Q.md
title: 20241006 - 阅读论文: Contextual retrieval
indexed_at: 2026-03-05T10:46:40.441699+00:00
---

## 摘要
介绍 Anthropic 的 Contextual retrieval 技术，核心是 Embedding+BM25 混合检索优于单独使用 Embedding。通过 LLM 为每个 chunk 生成上下文摘要，将 chunk 和完整文档输入 LLM 生成用于改善检索的上下文信息。

## 关键概念
- Contextual retrieval: 为文档切片生成上下文摘要以提升检索准确率的技术
- 混合检索: 结合 Embedding 语义检索和 BM25 关键字匹配的检索策略
- 上下文摘要: LLM 生成的用于连接 chunk 与完整文档的简洁上下文信息
- Chunk 情境化: 将切片置于完整文档语境中以便改进搜索召回

## 关联笔记
- 01KJBZFPM6BX07HW9FQM0QZ670.md: 同样探讨 chunk 处理和上下文增强 (AutoContext 自动上下文)
- 01KJBZG3NMEV9RPF9NN5VBTQ9Z.md: 讨论检索器优化，提及 BM25 与嵌入模型的检索效果对比
- 01KJBZR0R965XV1E3R1YF93FBG.md: 涉及多阶段检索策略，同样使用 BM25 进行初始召回
