---
note: 01KJBZ65Y2QFCFWKWQVM7T6NQ3.md
title: 20240211 - 对 公司DBAKnowledgeBase进行调优 [1] - 拆解问题要素
indexed_at: 2026-03-05T09:36:53.711263+00:00
---

## 摘要
记录使用 LLM 对 DBA 问题进行要素拆解的方法，通过 GPT-4/通义千问评估召回文档与各要素的相关性并进行 rerank。提出两种优化思路：要素拆解 + 评分公式，或将要素组织成抽象问题提高命中率（类似 STEP-BACK PROMPTING）。

## 关键概念
- 问题要素拆解: 将 DBA 问题抽象为关键搜索要素，用于提高文档召回准确性
- 文档相关性评分: 遍历每个要素，用 LLM 评估文档与要素的逻辑关系（直接相关/间接相关/无关）
- STEP-BACK PROMPTING: 将具体问题组织成更抽象的问题，提高召回文档命中率
- Rerank: 根据各要素评估结果对召回文档进行重新排序
- FAISS: 用于文档向量召回的相似度搜索库

## 关联笔记
- 01KJBZ5XTNYJSRXZ72WTVJ391G.md: STEP-BACK PROMPTING 论文阅读笔记，本笔记提到参考该方法组织抽象问题
- 01KJBZ64PQ0DMVAKYHABCZAV4K.md: 同系列前一天笔记，尝试对 DBAKnowledgeBase 输出进行评分
- 01KJBZAF5A9DS1CND4J22B9X6V.md: 通义千问 plus 和 max 差异测试，本笔记提到使用通义千问 API 进行评估
