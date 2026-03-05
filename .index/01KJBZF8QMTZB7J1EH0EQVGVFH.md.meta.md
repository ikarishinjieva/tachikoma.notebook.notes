---
note: 01KJBZF8QMTZB7J1EH0EQVGVFH.md
title: 20240907 - 阅读论文*: STRUCTURED-RAG: JSON RESPONSE FORMATTING WITH LARGE LANGUAGE MODELS
indexed_at: 2026-03-05T10:40:57.085690+00:00
---

## 标签
LLM, JSON 格式化，提示策略，StructuredRAG，基准测试，模型评估

## 摘要
论文提出 StructuredRAG 基准测试，包含六项任务评估 LLM 生成结构化 JSON 输出的能力。对比 f-String 和 Follow the Format (FF) 两种提示策略，发现 FF 策略在所有模型上均表现更优，Gemini 1.5 Pro 整体性能最佳。

## 关键概念
- StructuredRAG: 包含六项任务的基准测试框架，评估 LLM 结构化输出能力
- f-String 提示: 仅给出 JSON 结构模板的简洁提示方法
- Follow the Format (FF): 提供完整填充示例的两步式提示策略
- OPRO 优化: 针对特定任务的改进技术
- 多模型比较: 在 Gemini、GPT、Llama 等模型上验证方法有效性

## 关联笔记
- 01KJBZG071EGFZEW1XATH1JV1T.md: 同样讨论 LLM 提示策略优化的 REAP 论文笔记
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: RAG 系统优化相关的 Speculative RAG 论文笔记
- 01KJBZF7F960C9BRB1RD259XZM.md: Graph 与 RAG 结合场景的探索结论
