---
note: 01KJBYYZFEHS0QT21JG2ZJG2W9.md
title: 20230502 - llama-index + openai + mcontriever, 使用prompt解决信息示踪的问题
indexed_at: 2026-03-05T08:40:19.033260+00:00
---

## 标签
llama-index, 提示词工程, 信息溯源，RAG, refine, 知识库问答

## 摘要
记录使用 llama-index 配合自定义 prompt 实现答案溯源的两种方案，通过在 QA 和 Refine 模板中增加来源标注要求，使模型输出答案时明确标识每个部分来自上下文的哪一章节。展示了两次迭代的效果对比，第二次将"章"改为"部分"提升了引用精度。

## 关键概念
- QuestionAnswerPrompt: llama-index 的问答提示模板，用于初始化答案生成
- RefinePrompt: llama-index 的优化提示模板，用于多轮迭代合并答案
- 信息示踪: 在 RAG 系统中追踪答案各部分来源于哪个文档片段的技术
- similarity_top_k: 检索时返回的相似文档片段数量参数

## 关联笔记
- 01KJBYYVC6H5SK2N47CBNHJD5X.md: 前一天测试 mcontriever + openai + llama_index 的基础测试笔记
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 后续探索 llama_index 的 refine 机制为何未生效的问题分析
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 引用本笔记的溯源方案，用于构建私有运维知识库 demo
