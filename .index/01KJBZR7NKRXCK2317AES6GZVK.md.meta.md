---
note: 01KJBZR7NKRXCK2317AES6GZVK.md
title: 20250205 - 阅读论文*: RARE: Retrieval-Augmented Reasoning Enhancement for Large Language Models
indexed_at: 2026-02-27T11:31:06.932857+00:00
---

用户要求我分析这篇关于 RARE 论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：RARE 论文 (Retrieval-Augmented Reasoning Enhancement for Large Language Models)
2. 核心内容：
   - RARE 是基于 rStar 的 RAG 扩展
   - 节点扩展阶段：通过 A6 和 A7 动作引入 RAG
   - 判别步骤：使用 RAFS 判别器进行事实性评估
   - RAFS 的详细步骤说明

3. 标签应该是：RARE, RAG, LLM, 推理增强, RAFS, 事实性评估 等

4. 摘要需要概括核心内容

5. 关键概念：RARE, RAFS, RAG, 推理路径, 事实性评估等

6. 关联笔记：浏览其他笔记列表，都是关于数据库、MySQL、Clickhouse、系统性能等方面的笔记，与这篇 LLM/RAG 论文笔记没有明显关联

让我按要求生成输出。## 标签
RARE, RAG, LLM, 推理增强, RAFS, 事实性评估

## 摘要
RARE 是在 rStar 基础上引入 RAG 的推理增强框架，通过 A6/A7 动作在节点扩展阶段检索外部知识，并使用 RAFS 判别器评估推理路径的事实性。RAFS 通过将推理路径拆解为陈述句、生成多角度检索查询、收集正反证据来综合评分。

## 关键概念
- RARE: 基于 RAG 的推理增强框架，在 rStar 基础上扩展检索能力
- RAFS: 检索增强事实性评分器，通过外部知识评估推理路径真实性
- 节点扩展: 通过 A6(初始问题检索) 和 A7(子问题检索) 两个动作引入外部知识
- 事实性评估: 将推理路径拆分为陈述句，根据检索证据判断正误

## 关联笔记
无
