---
note: 01KJBZ044K0F6J2PM9E6FVJ07T.md
title: 20230523 - llama_index为什么不使用refine
indexed_at: 2026-03-05T08:44:43.396986+00:00
---

## 摘要
排查 llama_index 未使用 refine 的原因，发现是 response_mode 默认值 compact 导致多片段合并为单查询。增大 similarity_top_k 可触发 refine，但会导致 LLM 调用超出 max_token 限制。

## 关键概念
- response_mode: 控制查询引擎的响应合成模式，默认 compact 会合并片段
- compact: CompactAndRefine 模式，将多个文章片段压缩到一个 query 中
- similarity_top_k: 控制返回的文章片段数量，调大可触发 refine 但增加 token 消耗
- refine: 对多个检索片段进行迭代优化的响应合成方式
- max_token: LLM 单次调用的 token 上限，超出会抛出异常

## 关联笔记
- 01KJBYYZFEHS0QT21JG2ZJG2W9.md: 同系列 llama_index 笔记，涉及 RefinePrompt 提示词配置
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 同系列 llama_index 笔记，涉及 query_engine 和索引配置
- 01KJBYYVC6H5SK2N47CBNHJD5X.md: 同系列 llama_index 测试笔记，涉及 mcontriever 和 openai 集成
