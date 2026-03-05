---
note: 01KJBYZXHS7VYMQPF4FGZ6MMG1.md
title: 20230513 - 对langchain的MapReduceDocumentsChain机制分析
indexed_at: 2026-03-05T08:43:42.058303+00:00
---

## 摘要
分析 LangChain 的 MapReduceDocumentsChain 核心机制，描述其通过 _call 调用 combine_docs 处理文档数组的完整流程。核心是分层合并策略：先对每个文档调用 llm_chain，再按 tokens_max 分组，通过多轮_collapse_chain 将分组内文档合并为单个文档，最终由 combine_document_chain 合成统一输出。

## 关键概念
- MapReduceDocumentsChain: LangChain 中用于处理多文档的链式组件，采用分组合并策略
- combine_docs: 核心方法，对文档数组进行分层处理并合成最终结果
- llm_chain: 对每个文档单独调用 LLM 获取初步处理结果
- _collapse_chain: 将分组内的多个文档合并为单个文档的链式组件
- tokens_max: 控制文档分组的最大 token 数限制，决定合并轮次

## 关联笔记
- 01KJBYZYZDAJKTP18T9VCBWZ1M.md: 提到对文档切片后进行 map-reduce 处理时切口处信息丢失的问题
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 涉及 llama_index 的 CompactAndRefine 机制和 llm_chain 的 token 限制问题
