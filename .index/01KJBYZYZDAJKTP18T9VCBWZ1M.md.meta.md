---
note: 01KJBYZYZDAJKTP18T9VCBWZ1M.md
title: 20230514 - 制作私有运维知识库的demo (2)
indexed_at: 2026-03-05T08:43:55.229582+00:00
---

## 摘要
记录私有运维知识库 demo 开发进展，包括人工修订文档测试效果及发现的问题。重点分析文档预处理（摘要/重写）的难点和 llama_index 中 streaming 功能无法触发的技术问题。

## 关键概念
- 文档预处理: 对 Confluence 导出的文档进行 summary 和重写以提升检索效果
- Map-reduce: 文档切片后合并处理的策略，但存在章节层次信息丢失问题
- Streaming: 实时流式输出功能，在 tool 调用链中无法被监听 handler 捕获
- Token 限制: 大模型上下文长度限制导致长文档处理报错的问题

## 关联笔记
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 私有运维知识库 demo 的第一篇，包含初始代码和架构设计
- 01KJBYZ2DGB2PDS3ADP9JSCQ16.md: 涉及 llama_index 的 token 计算和查询日志分析
- 01KJBZ02F04VSM5G32S07EJAEJ.md: 后续对 opssage 项目的评估笔记
