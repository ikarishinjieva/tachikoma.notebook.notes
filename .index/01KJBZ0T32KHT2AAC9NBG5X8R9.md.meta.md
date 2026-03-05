---
note: 01KJBZ0T32KHT2AAC9NBG5X8R9.md
title: 20230608 - Text-To-SQL 的测试基准集 和 UnifiedSKG框架
indexed_at: 2026-03-05T08:47:22.592167+00:00
---

## 摘要
介绍 DAMO-ConvAI 测试基准集用于测试 text-to-SQL 准确度，包含 fine tuning 方法和多个 SQL 数据集。阐述 UnifiedSKG 框架通过 21 个预处理任务将结构化知识 (SQL/表格) 转换为文本供大模型处理。

## 关键概念
- DAMO-ConvAI: Text-To-SQL 测试基准集，用于评估模型准确度
- UnifiedSKG: 结构化知识统一处理框架，融合 21 个预处理任务
- SKG (Structured Knowledge Grounding): 结构化知识如 SQL、表格等
- T5 模型: 使用 UnifiedSKG 方法进行 fine tuning 的文本模型

## 关联笔记
- 01KJBZ19E24XJYJVVB725Q2YNT.md: 同日期笔记，讨论 SQL 引擎大模型选型和 spider 榜单，涉及 SQL 数据集和 T5 fine tuning
