---
note: 01KJBZ7KPW7MR951Y82WGFBVF7.md
title: 20240317 - 用Haystack实现ChatDBA[2] - 对对话进行评分
indexed_at: 2026-03-05T09:39:56.724547+00:00
---

## 摘要
这是 ChatDBA 系列第二篇，使用 Haystack 框架实现对话质量评分系统。定义了"突兀程度"指标，用于评估 AI 在对话中引入未提及客观事实的程度（微/低/中/高四级）。该评分系统为后续流水线调整提供稳定的质量评估标准。

## 关键概念
- 突兀程度: 评估 AI 使用之前对话未提及信息的突然性，分为微/低/中/高四级
- 对话评分系统: 对流水线输出质量进行评估的机制，用于稳定后续调整
- Haystack: 实现 ChatDBA 对话流水线的框架

## 关联笔记
- 01KJBZ7541N75M2EZ611NS9CMN.md: 用 Haystack 实现 ChatDBA 的第一篇，建立基础对话流水线框架
- 01KJBZGCGK8RMW6B39W70BRA25.md: 后续设计的 ChatDBA 问答质量评论系统方案
- 01KJBZGDRKYMRQVC8CKDG8NVNA.md: 训练问答质量评论系统的 SFT+DPO 实践
