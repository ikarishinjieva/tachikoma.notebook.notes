---
note: 01KJBZ7N24H4N9E6T7FCXAHYES.md
title: 20240323 - 关于LLM的function call的理解
indexed_at: 2026-03-05T09:40:38.917184+00:00
---

## 摘要
笔记描述了 LLM Function Call 的四步工作流程：通过提示词告知 LLM 可用函数、LLM 返回需调用的函数及参数、人类执行并返回结果、LLM 进行后续处理。参考了 Google Cloud Vertex AI 的官方文档。

## 关键概念
- Function Call: LLM 根据理解选择调用哪个函数并返回参数
- 提示词: 用于告知 LLM 存在哪些可调用的 function
- 人机协作: Human 负责执行 function 并返回结果给 LLM

## 关联笔记
- 01KJBZD3K42PBXXQ4KSTFQDP4N.md: 提到 OpenAIFunc 策略，利用 OpenAI 模型的函数调用能力将工具函数暴露给 LLM
- 01KJBZT45Z2MD7YR2QNH7HTJ4W.md: 同样参考 Google Cloud Generative AI 文档，涉及 Vertex AI 的视频生成 API 使用
