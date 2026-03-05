---
note: 01KJBZ9C1ZG41EB9NF0W87ZXSP.md
title: 20240520 - 阅读论文: AceCoder: Utilizing Existing Code to Enhance Code Generation
indexed_at: 2026-03-05T09:54:53.138535+00:00
---

## 摘要
AceCoder 通过 RAG 召回<需求，代码>对，并将其转换为<需求，中间结果，代码>三元组来增强代码生成。中间结果（如测试用例、API）辅助 LLM 理解代码用途，提升生成质量。论文还提出基于词匹配的去重算法处理召回结果。

## 关键概念
- 中间结果: 代码的效果表达（如测试用例、API），用于辅助 LLM 理解代码用途
- 三元组示例: <需求，中间结果，代码>格式，比传统<需求，代码>对包含更多语义信息
- 代码召回: 通过 RAG 从代码库中检索与当前需求相似的<需求，代码>对
- 去重算法: 基于词匹配置信度，去除召回结果中的重复代码片段

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，涵盖代码生成领域的 RAG 应用分类和方法
- 01KJBZD2RGYKMES9WAV22CF01C.md: ToolCoder 同样用 RAG 增强代码生成，但聚焦于 API 搜索工具调用
- 01KJBZD3K42PBXXQ4KSTFQDP4N.md: CODEAGENT 在代码生成全流程中集成检索工具，实现仓库级代码生成
