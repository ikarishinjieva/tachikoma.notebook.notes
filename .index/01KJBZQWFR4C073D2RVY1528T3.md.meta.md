---
note: 01KJBZQWFR4C073D2RVY1528T3.md
title: 20250130 - 阅读论文: BoT: Buffer of Thoughts: Thought-Augmented Reasoning with Large Language Models
indexed_at: 2026-03-05T11:15:12.954283+00:00
---

## 摘要
BoT 将 CoT 思考抽象为可复用的思想模板，通过问题蒸馏器、元缓冲区和缓冲区管理器三个组件协同工作。相比传统 CoT，BoT 具有更高的抽象程度、支持动态更新进化、可检索复用已有模板。

## 关键概念
- 问题蒸馏器：将复杂自然语言问题转化为结构化的、易于 LLM 理解的形式，提取关键信息和约束条件
- 元缓冲区：存储从过去问题解决过程中学习到的高级思想模板，支持检索和实例化
- 缓冲区管理器：从问题解决过程中提取新模板并动态更新元缓冲区，使系统不断进化
- 思想模板：解决某一类问题的通用思路和步骤，比 CoT 更具抽象性和泛化能力

## 关联笔记
- 01KJBZWP1711J31GDNMNTAQW01.md: 分析 CoT 推理中的熵模式，与 BoT 同属 LLM 推理优化方向
- 01KJBZETD71Z66AHRQNQ181WNG.md: Auto-CoT 自动生成 CoT 演示，与 BoT 的模板库构建理念相关
- 01KJBZPTXXKTXVDR009ADNWMAK.md: Coconut 在连续 latent space 中进行推理，与 BoT 同为 CoT 改进方案
