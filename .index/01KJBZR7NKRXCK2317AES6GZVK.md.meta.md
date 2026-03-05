---
note: 01KJBZR7NKRXCK2317AES6GZVK.md
title: 20250205 - 阅读论文*: RARE: Retrieval-Augmented Reasoning Enhancement for Large Language Models
indexed_at: 2026-03-05T11:22:36.745899+00:00
---

## 标签
RARE, RAG, LLM 推理, RAFS, 事实性评估, 思维链

## 摘要
RARE 在 rStar 基础上引入 RAG 扩展：节点扩展阶段通过 A6/A7 动作检索外部知识扩展思维素材，判别步骤用 RAFS 取代原判别器进行事实性评估。RAFS 通过拆分陈述、生成多角度查询、检索证据、评估支持/反驳比例来量化推理路径的真实性。

## 关键概念
- RARE: Retrieval-Augmented Reasoning Enhancement，在 rStar 基础上引入 RAG 增强 LLM 推理的框架
- RAFS: Retrieval-Augmented Factuality Scorer，利用外部知识库评估推理路径事实性的判别器
- A6/A7 动作: RARE 新增的检索动作，A6 针对初始问题检索，A7 针对子问题检索
- 陈述拆分: 将推理路径拆解为独立陈述句以实现细粒度事实性评估
- 多角度查询: 针对每个陈述生成支持和反驳两类查询以确保证据评估的全面性

## 关联笔记
- 01KJBZR3XNRAA8E4VR5EX8VSZM.md: rStar 是 RARE 的基础框架，RARE 在其基础上进行了 RAG 扩展并用 RAFS 替换了原判别器
