---
note: 01KJBZB5Z3B527PJ4RY1B179WE.md
title: 20240725 - 知识图谱的关系抽取[2]
indexed_at: 2026-03-05T10:25:52.657912+00:00
---

## 摘要
尝试使用 LLM 从 MySQL 官方文档中抽取实体并构建知识图谱关系图。通过固定实体方法生成 Mermaid 格式的关系图，展示 MySQL、子查询、存储函数等实体间的关联。探讨了该方法的查询局限性，计划参考 LlamaIndex Graph 模块和 CoE 论文改进方案。

## 关键概念
- 实体抽取: 从文本中识别并提取领域内的关键概念作为搜索关键字
- 关系图: 使用 Mermaid 格式可视化实体之间的语义关联
- 非确定性结果: 存储函数执行次数不确定导致的查询结果不一致问题

## 关联笔记
- 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md: 同系列前一篇，探讨知识图谱关系抽取的三元组建模方法
- 01KJBZBDWCWQHY5AD2FMAZEBFQ.md: 笔记中明确引用的 CoE 论文阅读笔记，介绍 KG-RAG 方法
- 01KJBZBMFN7QZYG0SXH1XM3S7Z.md: LlamaIndex Graph 模块分析，TODO 中提到的参考方案
