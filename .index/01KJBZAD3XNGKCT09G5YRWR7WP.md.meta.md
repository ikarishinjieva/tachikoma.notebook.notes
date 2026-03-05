---
note: 01KJBZAD3XNGKCT09G5YRWR7WP.md
title: 20240628 - 测试GENREAD (通过LLM生成"召回文档")
indexed_at: 2026-03-05T10:16:29.776031+00:00
---

## 摘要
记录 GENREAD 方法在 ChatDBA 场景下的测试过程，通过 LLM 生成技术合理的"召回文档"替代传统检索。展示了 MySQL crash 排查问题的两步生成流程及输出示例。

## 关键概念
- GENREAD: 用 LLM 生成上下文文档替代检索的 RAG 方法
- 召回文档：RAG 系统中检索或生成用于辅助回答的相关文档
- 思维链 (CoT): 引导 LLM 逐步推理生成结构化内容的提示技术

## 关联笔记
- 01KJBZAGWAND5W9R05TTT38871.md: 同系列笔记，记录在 ChatDBA 中测试 GENREAD 的后续实验
- 01KJBZ9PM9KV6T0DEC0GZ2B2H1.md: GENREAD 论文阅读笔记，记录该方法的核心思路
