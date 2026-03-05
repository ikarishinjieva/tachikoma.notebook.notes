---
note: 01KJBZ0ME2YJ6QRPQTGTQPXD0B.md
title: 20230527 - 吴恩达: 提示词工程课
indexed_at: 2026-03-05T08:45:34.828414+00:00
---

## 摘要
记录吴恩达提示词工程课程的核心技巧，包括使用输出样例规范格式、少样本提示保持一致风格、引导模型先推理后结论等方法。结合 OpenAI 官方文档建议，强调指令明确化、格式指定和逐步思考的重要性。

## 关键概念
- 输出样例: 提供期望输出格式的示例，引导模型按指定结构（如 Step 1-N）生成内容
- 少样本提示 (Few-shot): 通过样例让 LLM 学习并保持一致的回答风格
- 先梳理后结论: 提示模型先进行推理思考，再给出最终答案
- Redlines: 用于标记和可视化文本差异的工具

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: 对比反射标记与思维链 CoT，解释了"Let's think step by step"提示技巧的实现方式
- 01KJBZS20MPESNN97X6XPEZBV0.md: Chain of Draft 论文笔记，优化了"Think step by step"提示词，让模型只输出关键词句减少 token 消耗
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记，涉及 few-shot learning 在代码生成和检索增强中的应用场景
