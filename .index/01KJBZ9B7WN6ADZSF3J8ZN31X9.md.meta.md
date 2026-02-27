---
note: 01KJBZ9B7WN6ADZSF3J8ZN31X9.md
title: 20240516 - 阅读论文: REACT: SYNERGIZING REASONING AND ACTING IN LANGUAGE MODELS
indexed_at: 2026-02-27T10:53:34.743306+00:00
---

用户要求我分析这篇关于 ReAct 论文的笔记，生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：这是一篇关于 ReAct 论文的阅读笔记，ReAct 是一个结合推理（Reasoning）和行动（Acting）的语言模型框架。

2. **核心内容**：
   - 通过 few-shot 让 LLM 生成三种步骤：Search/Lookup/Finish
   - 提供了详细的提示词示例
   - 展示了如何在问答任务中交错使用 Thought、Action、Observation

3. **标签**：应该是关于 LLM、推理、Agent、ReAct、问答系统等

4. **关键概念**：
   - ReAct：结合推理和行动的语言模型框架
   - Few-shot prompting：通过示例引导模型
   - Search/Lookup/Finish：三种行动类型
   - Thought-Action-Observation： interleaving 的执行流程

5. **关联笔记**：查看其他笔记列表，大部分是关于 MySQL、Clickhouse、系统性能等，与这篇 LLM/ReAct 的笔记没有明显关联。唯一可能相关的是 "20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation" 但这篇也是关于数据库的，不是 LLM 相关。所以应该是「无」。

现在生成输出：## 标签
LLM, ReAct, 推理与行动, Few-shot Prompting, 问答系统, Agent

## 摘要
本笔记记录了 ReAct 论文的核心方法，通过 few-shot 提示让大语言模型交替生成推理（Thought）和行动（Action）。行动分为三种类型：Search 搜索实体、Lookup 查找关键词、Finish 返回答案，并结合 Observation 形成完整的问题解决流程。

## 关键概念
- ReAct: 将推理（Reasoning）与行动（Acting）协同结合的语言模型框架
- Few-shot Prompting: 通过示例引导模型学习任务模式
- Thought-Action-Observation: 交错执行的三步循环，实现逐步推理与外部交互
- Search/Lookup/Finish: 三种行动类型，分别用于搜索、查找和完成任务

## 关联笔记
无
