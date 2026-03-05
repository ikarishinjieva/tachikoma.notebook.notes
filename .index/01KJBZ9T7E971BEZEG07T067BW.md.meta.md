---
note: 01KJBZ9T7E971BEZEG07T067BW.md
title: 20240612 - 阅读论文: SARGAM: Automated Code Editing with Search-Generate-Modify
indexed_at: 2026-03-05T10:04:40.803852+00:00
---

## 摘要
SARGAM 将代码生成拆分为召回样例、生成初步代码、细粒度修改三步骤，核心创新是使用 LevT 类编辑操作模型进行代码修订。相比 LLM 直接修订，LevT 在局部修改任务中效率更高、修改更精准，但泛化能力较弱。

## 关键概念
- SARGAM: 搜索 - 生成 - 修改三阶段代码编辑框架
- LevT: 基于编辑操作（插入、删除、替换）的代码修改模型
- 编辑操作模型: 预测细粒度编辑操作而非从头生成完整代码序列的模型
- 代码召回: 从代码库中检索相似代码样例作为生成参考

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 该笔记汇总了多个代码生成相关论文，其中明确引用了本 SARGAM 笔记
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: 同为代码生成领域的论文笔记，讨论 SRA-MCTS 算法优化代码生成过程
- 01KJBZD3K42PBXXQ4KSTFQDP4N.md: CODEAGENT 论文笔记，探索代码生成全流程的多种创新策略
