---
note: 01KJBZ9RYTBE5YJST0JVNP925P.md
title: 20240607 - 阅读论文: FiD-Light: EFFICIENT AND EFFECTIVE RETRIEVAL-AUGMENTED TEXT GENERATION
indexed_at: 2026-03-05T10:03:00.880262+00:00
---

## 标签
FiD-Light, 检索增强生成, 向量压缩, 解码器优化, Source Pointers, RAG

## 摘要
FiD-Light 在 FiD 架构基础上，通过在解码阶段前压缩长向量来降低解码成本。引入 Source Pointers 机制，在向量中增加源文档标记并在解码器中保留，实现输出内容的可追溯性。

## 关键概念
- FiD 架构: 将问题和召回文档进行向量编码，将长向量放入解码器中生成答案，在解码过程中进行思考
- FiD-Light: 在 FiD 的解码阶段之前对长向量进行压缩，减少解码成本
- Source Pointers: 在向量中增加源文档标记，在解码器中保留标记，实现输出示踪

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文笔记，将 FiD 归类为基于表征的 RAG 典型实现，并引用了 FiD-Light 的压缩与示踪思路
