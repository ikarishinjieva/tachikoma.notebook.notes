---
note: 01KJBZ9B7WN6ADZSF3J8ZN31X9.md
title: 20240516 - 阅读论文: REACT: SYNERGIZING REASONING AND ACTING IN LANGUAGE MODELS
indexed_at: 2026-03-05T09:53:02.314003+00:00
---

## 标签
ReAct, 语言模型, 推理与行动, few-shot, 问答任务, Thought-Action-Observation

## 摘要
笔记记录了 ReAct 论文的核心方法，通过 few-shot 让 LLM 生成 Thought-Action-Observation 交替步骤来解决问答任务。定义了三种行动类型：Search（搜索维基百科实体）、Lookup（查找段落中关键词）、Finish（给出最终答案）。

## 关键概念
- ReAct 框架: 协同推理 (Reasoning) 和行动 (Acting) 的语言模型方法
- Thought-Action-Observation 循环: LLM 生成思考→执行行动→观察结果的迭代过程
- Search/Lookup/Finish: 三种行动类型，分别用于搜索实体、查找句子、完成任务
- few-shot prompting: 通过示例教会 LLM 遵循特定的推理和行动模式

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 该笔记在检索增强部分引用了本 ReAct 笔记，作为递归检索的典型案例
- 01KJBZ5SXG26R34S13HPP05SYY.md: 该笔记将 ReAct 与 CoT、Self-Ask、ITER-RETGEN 等方法进行对比分析
- 01KJBZD3K42PBXXQ4KSTFQDP4N.md: 该笔记提到 CODEAGENT 使用 ReAct 策略作为代码生成的多种代理策略之一
