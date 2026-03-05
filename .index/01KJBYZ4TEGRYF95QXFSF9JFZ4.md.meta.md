---
note: 01KJBYZ4TEGRYF95QXFSF9JFZ4.md
title: 20230505 - 制作私有运维知识库的demo
indexed_at: 2026-03-05T08:42:00.937382+00:00
---

## 摘要
记录使用 llama-index 构建私有运维知识库 demo 的实现过程，包括将索引嵌入 chatbot、显示引用章节/行号等功能。通过自定义 DirectAgent 和提示词优化实现信息示踪，使用行号标注解决大模型无法准确理解行号概念的问题。

## 关键概念
- llama-index: 用于构建 LLM 应用的框架，提供索引和查询能力
- DirectAgent: 自定义 agent，将请求直接转发给 index 而非使用 langchain 现有 chain
- 信息示踪: 通过提示词让大模型标注答案来源的文档和行号
- Prompt Helper: llama-index 中用于管理 prompt 长度的组件
- mcontriever: 用于多语言检索的 embedding 模型

## 关联笔记
- 01KJBYYZFEHS0QT21JG2ZJG2W9.md: 同系列笔记，详细讲解使用 prompt 解决信息示踪问题的方法
- 01KJBYYMHN0C5CA4ARZ21B6AAN.md: 前期测试笔记，记录 llama_index + Azure OpenAI 的基础配置
- 01KJBZ04YB9HSQ98XKQ8KF2SMF.md: 后续修复笔记，解决 llama_index refine 接口在连续 chunk 中的问题
