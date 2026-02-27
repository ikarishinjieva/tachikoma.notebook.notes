---
note: 01KJBYZ2DGB2PDS3ADP9JSCQ16.md
title: 20230503 - 测试方法"Hypothetical Document Embeddings" (HyDE)
indexed_at: 2026-02-27T10:28:27.245316+00:00
---

用户要求我分析这篇笔记并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于测试 HyDE (Hypothetical Document Embeddings) 方法的笔记
2. 主要内容是测试如何在 llama_index 和 langchain 中使用 HyDE 来改进向量搜索
3. 使用了一个具体的测试问题：如何动态调整 MySQL 的 buffer pool 大小
4. 包含了测试代码、API 日志等

关键主题：
- HyDE (Hypothetical Document Embeddings)
- 向量搜索/嵌入
- llama_index
- langchain
- MySQL buffer pool（作为测试用例）

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理 - 与 MySQL InnoDB 相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 - 与 MySQL InnoDB 相关
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 与内存管理相关（buffer pool 是内存相关）

现在生成索引：## 标签
HyDE, 向量搜索, llama_index, langchain, MySQL, 嵌入模型

## 摘要
记录测试 Hypothetical Document Embeddings (HyDE) 方法在私有文档库搜索中的应用。通过 MySQL buffer pool 调整问题作为测试用例，对比 llama_index 和 ChatGPT 生成的假设性答案形式，并实现 langchain 中的 HyDE 集成代码。

## 关键概念
- HyDE (Hypothetical Document Embeddings): 用 LLM 生成假设性答案作为查询嵌入，提升向量搜索相关性
- ServiceContext: llama_index 中配置 LLM 和嵌入模型的服务上下文
- HypotheticalDocumentEmbedder: langchain 中实现 HyDE 的嵌入包装器
- innodb_buffer_pool_size: MySQL InnoDB 缓冲池大小参数，用于动态调整内存分配

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同属 MySQL InnoDB 缓冲池与刷脏页相关研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同属 MySQL InnoDB 缓冲池与刷脏页相关研究
- 01KJBYDAF12RN18BECZG1QSTEC.md: 涉及内存分析，与 buffer pool 内存管理主题相关
