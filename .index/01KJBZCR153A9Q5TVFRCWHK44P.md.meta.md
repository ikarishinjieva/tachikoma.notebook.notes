---
note: 01KJBZCR153A9Q5TVFRCWHK44P.md
title: 20240803 - 阅读论文: THINK-ON-GRAPH: DEEP AND RESPONSIBLE REASONING OF LARGE LANGUAGE MODEL ON KNOWLEDGE GRAPH
indexed_at: 2026-02-27T11:10:02.574363+00:00
---

用户要求我分析这篇关于 THINK-ON-GRAPH 论文的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：论文阅读笔记，关于 THINK-ON-GRAPH (ToG) 算法
2. 核心内容：
   - LLM+KG vs LLM*KG 两种范式的区别
   - ToG 算法的三个主要阶段：初始化、探索、推理
   - ToG-R 算法的说明
   - 使用 beam search 在知识图谱上进行搜索

3. 标签应该包括：LLM、知识图谱、ToG 算法、推理、论文阅读等

4. 关键概念：
   - LLM+KG 范式
   - LLM*KG 范式
   - ToG 算法
   - ToG-R 算法
   - Beam Search

5. 关联笔记：查看其他笔记列表，都是关于数据库、MySQL、Clickhouse 等技术内容，与这篇 LLM+KG 的论文笔记没有明显关联。

让我生成符合要求的输出。## 标签
LLM, 知识图谱, ToG 算法, 推理范式, 论文阅读, Beam Search

## 摘要
论文提出 Think-on-Graph (ToG) 框架，将 LLM 与知识图谱深度耦合，由 LLM 动态决定 KG 搜索路径而非被动检索。相比传统"LLM+KG"范式，ToG 通过迭代探索关系和实体构建推理路径，ToG-R 则进一步用 LLM 预测关系并用 KG 验证以提升效率。

## 关键概念
- LLM+KG 范式: 从 KG 检索信息增强 prompt，LLM 仅作为翻译器，松耦合方式推理能力受限
- LLM*KG 范式: KG 与 LLM 在推理每步协同工作，形成紧密耦合的深度推理方式
- ToG 算法: LLM 引导在 KG 上进行 beam search，迭代探索关系和实体构建推理路径
- ToG-R 算法: LLM 预测下一步关系并用 KG 验证，目标导向更强但精度略低
- Beam Search: 类似带成本剪枝的广度优先搜索，每层保留 top-N 个候选路径

## 关联笔记
无
