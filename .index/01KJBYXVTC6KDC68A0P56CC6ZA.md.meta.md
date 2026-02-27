---
note: 01KJBYXVTC6KDC68A0P56CC6ZA.md
title: 20230417 - 测试chatglm-6b + llama_index
indexed_at: 2026-02-27T10:27:08.368304+00:00
---

用户希望我分析这篇关于测试 ChatGLM-6B 和 llama_index 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇技术笔记，关于测试 ChatGLM-6B 模型配合 llama_index 的使用
2. 记录了测试效果（中英文混杂问题、对dynamically理解不对）
3. 记录了TODO：测试另一个项目
4. 记录了诊断 warning 的过程，关于 token 序列长度超过模型限制的问题
5. 包含代码片段和调试过程

基于内容，我需要生成：
- 标签：3-6 个关键词
- 摘要：2-3 句话
- 关键概念：3-5 个
- 关联笔记：从提供的列表中找相关的

关于关联笔记，我看了看列表，大部分是关于 MySQL、Clickhouse、数据库相关的笔记，与这篇 AI/LLM 主题的笔记没有明显关联。所以关联笔记应该是「无」。

让我生成元数据：

标签：ChatGLM-6B, llama_index, 大语言模型, Token 限制, 调试

摘要：记录测试 ChatGLM-6B 配合 llama_index 的效果，发现中英文混杂和语义理解问题。通过堆栈追踪诊断 tokenizer 序列长度超限的 warning。

关键概念：
- ChatGLM-6B: 智源研究院开源的对话大语言模型


- llama_index: 用于构建 LLM 应用的索引框架
- Token 序列长度：模型处理输入的最大 token 数限制
- tokenizer: 将文本转换为模型可处理的 token 序列

关联笔记：无（其他笔记主要是数据库相关，与此 AI 主题无关）## 标签
ChatGLM-6B, llama_index, 大语言模型, Token 限制, 调试

## 摘要
记录测试 ChatGLM-6B 配合 llama_index 的效果，发现中英文混杂和语义理解问题。通过堆栈追踪诊断 tokenizer 序列长度超过模型最大限制（5446 > 2048）的 warning。

## 关键概念
- ChatGLM-6B: 智源研究院开源的对话大语言模型
- llama_index: 用于构建 LLM 应用的索引框架
- Token 序列长度: 模型处理输入的最大 token 数限制
- tokenizer: 将文本转换为模型可处理的 token 序列

## 关联笔记
无
