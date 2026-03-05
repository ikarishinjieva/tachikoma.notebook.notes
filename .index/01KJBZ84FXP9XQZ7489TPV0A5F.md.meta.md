---
note: 01KJBZ84FXP9XQZ7489TPV0A5F.md
title: 20240504 - ChatDBA: 预处理文档并生成高质量Plan, 需要进行论文查找
indexed_at: 2026-03-05T09:46:36.813478+00:00
---

## 摘要
ChatDBA 第一期发现 Plan 生成质量不稳定，需探索将文档改写成排查 Plan 的方法。调研 Prompt-RAG、QA-RAG 等论文，寻找预生成目录和 fake answer 召回的文档处理策略。

## 关键概念
- ChatDBA: 数据库故障诊断 AI 系统，通过生成排查 Plan 协助问题解决
- Plan 生成: 将文档信息转化为结构化排查步骤的过程，质量影响后续诊断稳定性
- Prompt-RAG: 放弃 Embedding，预生成"目录"进行文档召回的 RAG 方法
- QA-RAG: 用微调 LLM 生成 fake answer，结合原始 query 进行文档召回

## 关联笔记
- 01KJBZ98FFPHPW0FYMVDST5BCV.md: ChatDBA 第二期 Case-1 案例分析，尝试文档预处理生成高质量 Plan
- 01KJBZ83MN04S8N329FM6BMDXB.md: 前期改善 Plan 生成的尝试，将细节信息放在诊断前期
- 01KJBZ7TPWYT2ZSTJ3GDM1APY3.md: 讨论 Plan 重新生成的时机判断问题
