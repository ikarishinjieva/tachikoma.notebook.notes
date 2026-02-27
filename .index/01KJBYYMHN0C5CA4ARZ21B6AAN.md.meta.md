---
note: 01KJBYYMHN0C5CA4ARZ21B6AAN.md
title: 20230423 - 测试 llama_index + openai/azure
indexed_at: 2026-02-27T10:27:42.163272+00:00
---

用户要求我分析一篇关于测试 llama_index + openai/azure 的笔记，并生成索引元数据。我需要：

1. 提取 3-6 个标签
2. 写 2-3 句话的摘要
3. 列出 3-5 个关键概念
4. 找出 0-3 篇关联笔记

让我分析笔记内容：
- 测试 Azure OpenAI 服务
- 使用 llama_index 框架
- 测试了 text-embedding-ada-002 和 text-davinci-003 模型
- 测试了 gpt-35-turbo-0301 模型，遇到兼容性问题
- 分析了 LLM token 和 embedding token 的计算方式
- 诊断了 token 序列长度超限的 warning

标签应该是：Azure OpenAI, llama_index, 嵌入模型, LLM, token 计数, 模型测试

摘要：记录使用 llama_index 框架测试 Azure OpenAI 服务的实验过程和结果。对比了 text-davinci-003 和 gpt-35-turbo-0301 模型的效果，并分析了 token 计算机制和序列长度警告问题。

关键概念：
- text-embedding-ada-002: OpenAI 的文本嵌入模型
- text-davinci-003: OpenAI 的 LLM 模型
- llama_index: 用于构建 LLM 应用的索引框架
- token 计数: LLM 和 embedding 的 token 使用量统计


- 序列长度限制：模型输入的最大 token 数

关联笔记方面，这篇笔记聚焦于 Azure OpenAI 和 LLM 测试，而现有笔记主要是数据库和系统性能相关内容，没有直接关联的主题。## 标签
Azure OpenAI, llama_index, 嵌入模型, LLM, token 计数, 模型测试

## 摘要
记录使用 llama_index 框架测试 Azure OpenAI 服务的实验过程和结果。对比了 text-davinci-003 和 gpt-35-turbo-0301 模型的效果，并分析了 token 计算机制和序列长度警告问题。

## 关键概念
- text-embedding-ada-002: OpenAI 的文本嵌入模型，用于生成向量表示
- text-davinci-003: OpenAI 的 LLM 模型，效果优于 gpt-35-turbo
- llama_index: 用于构建 LLM 应用的索引框架，包含 query/retrieve/synthesize 流程
- token 计数: LLM token 为 prompt 加预测结果长度，embedding token 为输入内容长度
- 序列长度限制: tokenizer 的 model_max_length 限制，超长会导致索引错误

## 关联笔记
无
