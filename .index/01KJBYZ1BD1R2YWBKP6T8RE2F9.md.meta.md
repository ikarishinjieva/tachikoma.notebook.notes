---
note: 01KJBYZ1BD1R2YWBKP6T8RE2F9.md
title: 20230502 - 尝试openai的fine tuning
indexed_at: 2026-03-05T08:40:39.750435+00:00
---

## 标签
OpenAI, fine-tuning, 分类问题, logprobs, 数据集采集, 条件生成

## 摘要
记录 OpenAI fine-tuning 官方文档的核心场景：分类问题（处理模型错误陈述）和条件生成。提出用 ChatGPT 生成问题、人工标记答案的数据采集方案来解决测试数据不足的问题。

## 关键概念
- fine-tuning: 通过特定数据集微调模型以适配特定任务
- logprobs: 输出 token 的对数概率，用于评估模型置信度
- 分类问题: 将不正确陈述归类为"Supported: no"以排斥错误答案
- 条件生成: 基于特定条件控制模型输出内容
- 数据集采集: 用 ChatGPT 生成问题 + 人工标记的高效数据收集方法

## 关联笔记
- 01KJBZJ4ZJYX6CE5DC0AKEKGHH.md: 记录 Qwen2-VL 模型的 LoRA 微调训练实践
- 01KJBZEB2YCF8V8XY13DVWPEKZ.md: 讨论微调大模型成本高，提出用小模型生成 prompt 替代方案
