---
note: 01KJBZBDWCWQHY5AD2FMAZEBFQ.md
title: 20240725 - 阅读论文: CoE: KG-RAG: Bridging the Gap Between Knowledge and Creativity
indexed_at: 2026-03-05T10:26:12.368469+00:00
---

## 摘要
笔记记录了 CoE: KG-RAG 论文的两个核心创新点：三元组超节点构建知识图谱、以及由 LLM 调度的"规划 - 查找 - 评估"循环图搜索算法。三元组超节点包含更丰富的语义信息，CoE 通过迭代式探索逐步缩小搜索范围以找到与查询最相关的图谱信息。

## 关键概念
- 三元组超节点: 包含完整句子或多个三元组的复合节点，比传统三元组包含更多语义信息
- CoE 算法: 由 LLM 调度的循环式图搜索框架，执行"规划 - 查找 - 评估"三步迭代
- 规划 (Planning): 根据用户查询制定探索知识图谱的步骤顺序和查找目标
- KG Lookups: 结合向量相似性搜索和图数据库查询两种方式查找相关节点或关系
- 评估 (Evaluation): 判断检索信息是否满足需求，不满足则调整探索计划

## 关联笔记
- 01KJBZB5Z3B527PJ4RY1B179WE.md: 同天创建的知识图谱关系抽取笔记，明确引用了 CoE 论文作为参考
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 论文笔记，同为 KG-RAG 方向，使用 PageRank 进行图搜索
- 01KJBZBMFN7QZYG0SXH1XM3S7Z.md: LlamaIndex 知识图谱增强 RAG 流程分析，同属图谱检索增强主题
