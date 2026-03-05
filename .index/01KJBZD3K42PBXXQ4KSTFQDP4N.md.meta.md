---
note: 01KJBZD3K42PBXXQ4KSTFQDP4N.md
title: 20240805 - 阅读论文: CODEAGENT: Enhancing Code Generation with Tool-Integrated Agent Systems for Real-World Repo-level Codin
indexed_at: 2026-03-05T10:29:26.311288+00:00
---

## 摘要
CODEAGENT 论文提出了一套完整的仓库级代码生成方案，通过在需求理解、策略选择、代码实现和验证阶段引入创新设计，实现了从独立代码单元生成到代码仓库级别生成的跨越。系统支持多种代理策略（规则型、ReAct、Tool-Planning、OpenAIFunc），并深度集成信息检索、代码分析、测试等工具。

## 关键概念
- CODEAGENT: 工具集成的代理系统，用于真实场景的仓库级代码生成
- 规则型策略: 按预定义规则顺序调用工具，如先搜索文档再生成代码
- ReAct 策略: 让 LLM 动态决定何时调用哪些工具及如何整合输出
- Tool-Planning 策略: 先让 LLM 制定行动计划，明确工具调用顺序后执行
- 仓库级生成: 分析整个代码仓库信息，生成符合整体风格规范的代码

## 关联笔记
- 01KJBZD2RGYKMES9WAV22CF01C.md: ToolCoder 同样研究代码生成模型与 API 搜索工具的集成，但采用微调方式将工具使用知识植入模型
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: SRA-MCTS 同样针对代码生成任务，使用 MCTS 优化文本推理过程而非工具集成策略
- 01KJBZG6ZPGE03511EZ5WVFPQP.md: 工具辅助的 SQL 生成与纠错，同样采用工具增强 LLM 生成质量的思路
