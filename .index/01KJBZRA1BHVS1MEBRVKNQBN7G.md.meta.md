---
note: 01KJBZRA1BHVS1MEBRVKNQBN7G.md
title: 20250206 - 阅读论文*: FuncEvalGMN: Evaluating Functional Correctness of SQL via Graph Matching Network
indexed_at: 2026-03-05T11:24:30.820870+00:00
---

## 标签
SQL, 图匹配网络, GMN, 功能等价性, 图嵌入, 程序图

## 摘要
FuncEvalGMN 将 SQL 转换成包含逻辑流和数据流的程序图，通过 GMN 的消息传递和交叉注意力机制生成图嵌入，用于比较 SQL 的功能等价性。该 embedding 模型还可应用于 SQL 优化、索引推荐、查询计划缓存等场景。

## 关键概念
- 程序图：SQL 的关系运算树加上逻辑流和数据流边，形成包含完整语义的图结构
- 节点嵌入：包含节点特征嵌入（操作符、表名、列名等）和位置嵌入（结构信息）
- 图匹配网络 (GMN)：通过消息传递和交叉注意力机制将程序图聚合为图级向量表示
- 交叉注意力：捕捉两个图之间节点的对应关系，用于比较 SQL 的功能相似性
- 消息传递：节点与邻居之间传播特征信息，多轮传递后捕捉局部结构信息

## 关联笔记
- 01KJBZVR7G5C69KG304370BJ9D.md: 讨论 GRPO 训练中 SQL 优化的奖励函数设计，与 SQL 功能评估相关
- 01KJBZZJ92R5GJ0VH1J4W3X93K.md: SQL 优化的 GRPO 训练数据生成过程，涉及 SQL 语义分析
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 继续进行 SQL 优化的 GRPO 训练，探索 SQL 语义理解
