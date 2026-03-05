---
note: 01KJBYXVTC6KDC68A0P56CC6ZA.md
title: 20230417 - 测试chatglm-6b + llama_index
indexed_at: 2026-03-05T08:35:35.298102+00:00
---

## 摘要
记录 ChatGLM-6B 与 llama_index 集成的测试效果，发现中英文混杂和 token 超限问题。通过 traceback 堆栈追踪定位 tokenizer 调用位置，诊断序列长度超过模型最大限制 (5446 > 2048) 的警告。

## 关键概念
- ChatGLM-6B: 智源开源的 6B 参数中英双语大语言模型
- llama_index: 大模型应用的文档索引和检索框架
- Token 限制：模型单次处理的最大 token 数（ChatGLM-6B 为 2048）
- Traceback: Python 调试工具，用于打印函数调用堆栈

## 关联笔记
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 同系列 llama_index 调试笔记，探讨 response_mode 和 refine 机制
- 01KJBYYSWT0HJ1ZYSW82YFETXD.md: llama_index 索引结构和知识获取架构分析
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 使用 llama_index 构建私有知识库的实战 demo
