---
note: 01KJBYXRTFKTV6KQWHN0ZVT2KY.md
title: 20230405 - 测试 llama-index + GPT4All
indexed_at: 2026-03-05T08:31:52.861660+00:00
---

## 标签
llama-index, GPT4All, LangChain, 本地模型，文档问答，pyllamacpp

## 摘要
记录了使用 llama-index 结合 GPT4All 本地模型进行文档问答的测试过程，包含环境配置、模型转换和代码示例。测试中发现 token 长度限制导致的报错问题，最终未能解决。

## 关键概念
- llama-index: LLM 与外部数据连接的接口框架，提供数据结构索引大量数据
- GPT4All: 本地运行的开源语言模型，基于 llama.cpp 的 C/C++ 实现
- LangChain: LLM 应用开发框架，提供与多种语言模型集成的模块
- pyllamacpp: llama.cpp 的 Python 绑定库，支持 GPT4All 模型调用
- LLMPredictor: LangChain 中 LLMChain 的包装器，用于 llama-index 集成

## 关联笔记
- 01KJBYXQY4P60F03GGHAEDA2SW.md: 早期的 llama-index 试用笔记，时间上在此次测试之前
- 01KJBYXSMCBD001148DZFS1EGF.md: 后续测试 llama-index 与 OpenAI 的结合，时间上在此次测试之后
- 01KJBYXT9H9NANJAR8WY5T6NZ2.md: 测试 Cohere 与 llama-index 结合，探索不同模型方案
