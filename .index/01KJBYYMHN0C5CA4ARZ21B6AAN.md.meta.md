---
note: 01KJBYYMHN0C5CA4ARZ21B6AAN.md
title: 20230423 - 测试 llama_index + openai/azure
indexed_at: 2026-03-05T08:38:49.576982+00:00
---

## 摘要
记录使用 llama_index 集成 Azure OpenAI 服务的测试过程，对比 text-davinci-003 和 gpt-35-turbo-0301 模型效果。分析 LLM token 和 embedding token 的计算逻辑，诊断 token 序列超长 warning 问题。

## 关键概念
- LLM token usage: prompt 长度加上预测返回结果的长度总和
- embedding token usage: embed 操作前输入内容的长度
- text-davinci-003: OpenAI 完成类模型，支持 logprobs 等参数
- gpt-35-turbo-0301: Azure 聊天类模型，不支持 logprobs/best_of/echo 参数
- chunk_size_limit: llama_index 中控制文本分块大小的配置参数

## 关联笔记
- 01KJBYYVC6H5SK2N47CBNHJD5X.md: 同系列测试笔记，使用 mcontriever + openai + llama_index 组合
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 后续研究 llama_index 为何不使用 refine 机制
- 01KJBYY15XHA2SXQJ1QNJY255A.md: 同期测试 vicuna-13b + llama_index 的对比实验
