---
note: 01KJBZRC09Y66R17S5HP38G0TC.md
title: 20250211 - 阅读论文*: Why Are My Prompts Leaked? Unraveling Prompt Extraction Threats in Customized Large Language Models
indexed_at: 2026-03-05T11:26:52.367259+00:00
---

## 摘要
论文揭示 LLM 即使经过安全对齐仍容易受到提示提取攻击，其自注意力机制可构建从提示到生成文本的直接连接路径导致"平行翻译"现象。提出两大类防御策略：增加提示困惑度（随机插入、高困惑度改写）和阻断注意力链接（局部查找、重复前缀、假提示），可显著降低泄露率而不影响模型性能。

## 关键概念
- 平行翻译 (Parallel Translation): LLM 的某些注意力头直接将提示中的 token 复制到生成文本中的现象
- 提示困惑度 (Prompt Perplexity): 衡量提示对 LLM 熟悉程度的指标，低困惑度提示更容易被泄露
- 显式/隐式意图攻击: 直接要求透露提示 vs 通过巧妙设计诱导 LLM 泄露提示的攻击方式
- SPLIt 指标: 用于揭示 LLM 注意力机制中从提示到生成文本直接连接路径的测量指标

## 关联笔记
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 同样分析 LLM 内部注意力机制和思维追踪，使用归因图揭示模型内部计算路径
