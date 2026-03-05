---
note: 01KJBZ64ZMZGNC5XK5WNQCS17M.md
title: 20240210 - 对 公司DBAKnowledgeBase的代码分析
indexed_at: 2026-03-05T09:36:34.334985+00:00
---

## 摘要
分析公司 DBAKnowledgeBase 项目的代码结构，包括 Reason-Plan-Method 三阶段诊断流程。详细记录了各 Agent 的文档召回策略、重排序方法（RRF、RagasRerank、Cohere）和提示词设计。

## 关键概念
- ReasonAgent: 分析问题产生原因，使用通用 Retriever+RRF 重排序
- PlanAgent: 生成排查计划树，输出 Mermaid 格式流程图
- MethodAgent: 按排查树节点逐步引导用户，支持多轮对话
- ContextualCompressionRetrieverWrapper: 将召回文档每 5 个合并成一个
- 排查树: 用 Mermaid 图表示的诊断步骤结构

## 关联笔记
- 01KJBZ64PQ0DMVAKYHABCZAV4K.md: 同一天对 DBAKnowledgeBase 输出进行 GPT-4 评分的尝试
- 01KJBZ65Y2QFCFWKWQVM7T6NQ3.md: 次日对 DBAKnowledgeBase 进行调优，拆解问题要素
