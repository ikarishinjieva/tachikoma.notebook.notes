---
note: 01KJBYYVC6H5SK2N47CBNHJD5X.md
title: 20230501 - 测试 mcontriever + openai + llama_index
indexed_at: 2026-03-05T08:39:55.770821+00:00
---

## 摘要
测试 HuggingFaceInstructEmbeddings 和 HuggingFaceEmbeddings(mcontriever) 两种 embedding 模型在 llama_index 中的效果，验证中文问题检索英文 MySQL 文档的可行性。发现 query 接口默认 similarity_top_k=1 导致上下文不足，且模型会产生文档中不存在的幻觉信息（如 restore-from-image 命令）。

## 关键概念
- HuggingFaceEmbeddings: 用于文本向量化的嵌入模型，影响检索效果
- similarity_top_k: 控制检索返回的相似节点数量，默认值为 1
- 幻觉问题: 大模型生成文档中不存在的信息，需通过来源标记和负向反馈抑制

## 关联笔记
- 01KJBYYZFEHS0QT21JG2ZJG2W9.md: 次日继续测试，使用自定义 prompt 模板解决答案来源示踪问题
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 后续深入分析 llama_index 的 refine 机制和 response_mode 配置
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 基于相同技术栈的私有运维知识库 demo 项目
