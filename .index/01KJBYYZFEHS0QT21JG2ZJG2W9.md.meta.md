---
note: 01KJBYYZFEHS0QT21JG2ZJG2W9.md
title: 20230502 - llama-index + openai + mcontriever, 使用prompt解决信息示踪的问题
indexed_at: 2026-02-27T10:28:13.911329+00:00
---

用户要求我分析这篇笔记并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：使用 llama-index + openai + mcontriever，通过 prompt 解决信息溯源/追踪的问题
2. 核心内容：展示了两个版本的提示词模板，用于在 RAG 系统中追踪答案来源
3. 技术栈：llama-index, prompt engineering, 信息溯源
4. 场景：mysqlbackup 备份恢复的问答示例

关键概念：
- llama-index 的 QuestionAnswerPrompt 和 RefinePrompt
- 信息溯源/追踪（答案来源标注）
- 迭代式问答优化

关联笔记：从列表中看，这篇笔记主要是关于 RAG 和 prompt 工程的，与其他 MySQL 技术笔记的关联性不强。可能有一些间接关联，比如 MySQL 相关的笔记，但这篇笔记的核心是 prompt 工程而非 MySQL 本身。

让我生成索引：

标签：llama-index, prompt 工程，信息溯源，RAG, 问答系统

摘要：记录使用 llama-index 框架通过自定义 prompt 模板实现答案来源追踪的方法。展示了 QA 和 Refine 两种提示词模板的迭代优化过程，以 mysqlbackup 恢复操作为例演示效果。

关键概念：
- QuestionAnswerPrompt: llama-index 中用于生成初始答案的提示词模板
- RefinePrompt: 用于多轮迭代优化答案的提示词模板


- 信息溯源：在答案中标注每个部分来自上下文的哪个位置

关联笔记：无直接关联，因为这篇笔记重点是 prompt 工程方法论，而非 MySQL 技术细节。## 标签
llama-index, prompt 工程，信息溯源，RAG, 问答系统，上下文追踪

## 摘要
记录使用 llama-index 框架通过自定义 prompt 模板实现答案来源追踪的方法。展示了 QA 和 Refine 两种提示词模板的迭代优化过程，以 mysqlbackup 恢复操作为例演示多轮迭代中来源标注的效果。

## 关键概念
- QuestionAnswerPrompt: llama-index 中用于根据上下文生成初始答案的提示词模板
- RefinePrompt: 用于多轮迭代中合并新上下文信息并优化现有答案的提示词模板
- 信息溯源：在答案末尾标注每个部分来自上下文的哪个章节或文档

## 关联笔记
- 无
