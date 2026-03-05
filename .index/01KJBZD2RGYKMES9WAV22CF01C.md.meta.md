---
note: 01KJBZD2RGYKMES9WAV22CF01C.md
title: 20240804 - 阅读论文*: ToolCoder: Teach Code Generation Models to use API search tools
indexed_at: 2026-03-05T10:29:07.891019+00:00
---

## 摘要
ToolCoder 通过微调让代码生成模型学会何时调用 API 搜索工具，解决生成代码中 API 调用不准确的问题。采用三阶段方法：数据标注（用 ChatGPT 生成 API 搜索标注）、参数高效微调（LoRA 技术）、推理时工具增强（RAG 方式完善代码）。

## 关键概念
- ToolCoder: 集成 API 搜索工具的代码生成模型，通过微调学会何时查询 API
- LoRA: 低秩适应技术，用于参数高效微调，只训练少量参数降低成本
- API 搜索标注: 格式为 `<API>APISearch(query)->answer</API>` 的代码标注方式
- 推理时工具增强: 生成代码时动态调用搜索工具获取 API 信息并插入代码

## 关联笔记
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: 同为代码生成方向的论文笔记（SRA-MCTS），都关注提升代码生成质量
