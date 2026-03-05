---
note: 01KJBZEGMXBTY2YG5REW3607PM.md
title: 20240820 - 阅读论文: Ask me anything: A simple strategy for prompting language models
indexed_at: 2026-03-05T10:35:58.127361+00:00
---

## 摘要
AMA 方法通过多个提示词链的综合概率计算，抑制微小提示词改动导致的输出大幅波动。核心思路是对输入输出进行多样化改写，利用依赖图分析提示词链正交性，生成综合概率图得出最终输出。

## 关键概念
- 提示词链：包含 question() 和 answer() 两个 prompt 的固定或随机组合
- 输入改写：将原始输入改写成是非题、选择题、填空题等不同类型问句
- 输出改写：引导模型生成不同风格答案并映射到任务输出空间
- 概率图：综合所有提示词链能力得出的输出概率分布
- 依赖图：分析各提示词链正交性以调节概率图

## 关联笔记
- 01KJBZEDWSXQM2CW9H1ADKN1DQ.md: 同为 LLM 提示词策略相关的论文阅读笔记
- 01KJBZ7SDESPWSX3VZV2NMJD8G.md: 提示词技巧的通用方法论笔记
