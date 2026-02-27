---
note: 01KJBZ7N24H4N9E6T7FCXAHYES.md
title: 20240323 - 关于LLM的function call的理解
indexed_at: 2026-02-27T10:49:00.623340+00:00
---

用户要求我分析一篇关于 LLM function call 的笔记，并生成索引元数据。让我仔细分析笔记内容：

笔记标题：20240323 - 关于 LLM 的 function call 的理解

笔记内容主要是关于 LLM function call 的工作流程：
1. 通过提示词告诉 LLM 存在一些 function
2. LLM 根据理解告诉 Human 需要调用哪个 function 及参数
3. Human 告诉 LLM function 的执行结果
4. LLM 进行后续处理

参考文档是 Google Cloud 的 function calling 文档。

其他笔记列表主要是关于 MySQL、Clickhouse、数据库、性能调优等技术内容，与 LLM function call 没有直接关联。

现在我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个核心概念
4. 关联笔记：0-3 篇（看起来没有直接关联的）

让我生成这些内容：

标签：LLM, Function Call, 提示词，大语言模型，AI 交互

摘要：介绍 LLM Function Call 的工作机制和四步流程。通过提示词告知 LLM 可用函数，由 LLM 决定调用哪个函数并返回参数，人类执行后反馈结果给 LLM 继续处理。

关键概念：
- Function Call: LLM 根据理解选择调用特定函数并返回参数


- 提示词：用于告知 LLM 存在哪些可用函数的输入
- Human-LLM 交互：人类执行函数并反馈结果给 LLM 的协作模式

关联笔记：无（其他笔记都是关于数据库、MySQL、Clickhouse 等，与 LLM function call 无关）## 标签
LLM, Function Call, 大语言模型，提示词，AI 交互

## 摘要
介绍 LLM Function Call 的工作机制和四步交互流程。通过提示词告知 LLM 可用函数，由 LLM 决定调用哪个函数并返回参数，人类执行后反馈结果给 LLM 继续处理。

## 关键概念
- Function Call: LLM 根据理解选择调用特定函数并返回参数
- 提示词：用于告知 LLM 存在哪些可用函数的输入
- Human-LLM 交互：人类执行函数并反馈结果给 LLM 的协作模式

## 关联笔记
无
