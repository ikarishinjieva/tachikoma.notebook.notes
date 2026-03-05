---
note: 01KJBYYB3AQZY7FMQ8AHZGV7MY.md
title: 20230420 - koala-13b + llama_index 测试
indexed_at: 2026-03-05T08:38:30.759137+00:00
---

## 摘要
测试 koala-13b 模型配合 llama_index 进行 SQL 问答的效果。模型在 SET SESSION 语法上给出错误答案，且无法识别 enlarge 命令，整体表现不理想。

## 关键概念
- koala-13b: 基于 LLaMA 微调的 130 亿参数开源对话模型
- llama_index: 大模型应用开发框架，用于构建 RAG 系统
- RAG: 检索增强生成，结合外部知识库增强模型回答能力

## 关联笔记
- 01KJBYY15XHA2SXQJ1QNJY255A.md: 同一天测试 vicuna-13b + llama_index，同样存在无法识别 enlarge 的问题
- 01KJBYYMHN0C5CA4ARZ21B6AAN.md: 20230423 测试 llama_index + OpenAI/Azure，对比不同 LLM 的效果
