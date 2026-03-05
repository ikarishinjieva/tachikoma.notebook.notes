---
note: 01KJBZFPM6BX07HW9FQM0QZ670.md
title: 20240922 - 分析 dsRAG 项目 (对chunking的创新)
indexed_at: 2026-03-05T10:42:26.377816+00:00
---

## 摘要
分析 dsRAG 项目的文档处理流程，核心创新在于语义切片 (semantic sectioning) 与 AutoContext 自动上下文生成技术。文档先进行语义切片，再对整体和切片分别生成标题/摘要，最后将切片拆分为带描述头的 chunk 用于检索。

## 关键概念
- Semantic Sectioning: 将文档按语义结构自动切分为逻辑区段
- AutoContext: 自动生成文档/切片级别的标题和摘要
- Relevant Segment Extraction: 提取与查询相关的区段
- Chunk Header: 为每个 chunk 添加描述性头部信息

## 关联笔记
- 01KJBZR2HN0TS4QF7CGPMAZVXR.md: 讨论 chunking 中词的 embedding 向量池化方法
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 对召回文档进行聚类采样的类似思路
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 从知识图谱中定位段落的检索方法
