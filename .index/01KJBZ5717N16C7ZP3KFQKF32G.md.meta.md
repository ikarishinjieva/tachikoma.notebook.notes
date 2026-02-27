---
note: 01KJBZ5717N16C7ZP3KFQKF32G.md
title: 20240202 - 阅读论文: 综述: Retrieval-Augmented Generation for Large Language Models
indexed_at: 2026-02-27T10:42:55.290628+00:00
---

用户希望我分析这篇关于 RAG（Retrieval-Augmented Generation）论文阅读的笔记，并生成索引元数据。让我分析笔记内容：

1. 笔记主题：RAG 综述论文的阅读理解笔记
2. 核心内容：
   - RAG 的定义和框架（Naive RAG、Advanced RAG、Modular RAG）
   - RAG 各组件（检索器、生成器）
   - RAG 与微调的比较
   - 相关技术（滑动窗口、细粒度分割、元数据优化、REALM、RETRO 等）

3. 标签应该包括：RAG、大语言模型、信息检索、论文阅读等

4. 摘要：概括笔记核心内容

5. 关键概念：RAG、Naive RAG、Advanced RAG、Modular RAG、检索器、生成器等

6. 关联笔记：查看提供的笔记列表，主要是关于 MySQL、Clickhouse、系统性能等技术笔记，与 RAG/LLM 主题没有直接关联，所以应该是「无」

让我生成输出：## 标签
RAG, 大语言模型, 信息检索, 论文阅读, 生成式 AI

## 摘要
该笔记记录了 RAG（检索增强生成）综述论文的核心内容，涵盖 RAG 框架的三种研究范式演进：Naive RAG、Advanced RAG 和 Modular RAG。笔记整理了 RAG 与微调的对比、各范式的方法与优缺点，以及滑动窗口、细粒度分割、REALM、RETRO 等关键技术。

## 关键概念
- RAG（检索增强生成）: 结合检索阶段和生成阶段的模型架构，通过检索外部知识库增强生成能力
- Naive RAG: 早期 RAG 范式，通过编码模型检索文档后生成回复，存在检索限制和一致性问题
- Advanced RAG: 引入预检索和后检索方法优化索引质量，采用滑动窗口、细粒度分割等技术
- Modular RAG: 将 RAG 系统划分为多个模块，每个模块负责不同任务，更具灵活性和可扩展性
- 检索器与生成器: RAG 的两大核心组件，检索器负责语料库检索，生成器负责后处理检索文档并生成结果

## 关联笔记
无
