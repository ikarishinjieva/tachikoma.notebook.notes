---
note: 01KJBZE0C1T7XST2M7PBZYEV9C.md
title: 20240812 - 阅读论文: Intent-based Prompt Calibration: Enhancing prompt optimization with synthetic boundary cases
indexed_at: 2026-03-05T10:31:18.709289+00:00
---

## 标签
提示词优化, 边界案例合成, 迭代校准, LLM 评估, 论文阅读

## 摘要
该论文提出基于意图的提示校准 (IPC) 方法，通过迭代生成合成边界数据、评估提示性能、分析错误案例来优化提示词。核心创新是使用 LLM 生成对抗性边界案例，并在每轮迭代中根据错误分析生成新提示，逐步提升 LLM 在目标任务上的性能。

## 关键概念
- IPC (Intent-based Prompt Calibration): 基于意图的提示校准框架，通过迭代优化循环生成高质量提示词
- 合成边界数据: 使用 LLM 生成的具有挑战性的对抗性样本，用于暴露提示词弱点
- 迭代优化循环: 生成数据→评估性能→分析错误→生成新提示的四步循环流程

## 关联笔记
- 01KJBZDYTJF2N82HT70Z06VNQN.md: AutoPrompt 项目架构笔记，详细描述了与 IPC 一致的优化流程和提示词生成逻辑
- 01KJBZSKZ0GFCFP50QGNNNFB3Y.md: PROMPTCOT 论文笔记，同样使用 LLM 合成数据的方法来提升模型能力
