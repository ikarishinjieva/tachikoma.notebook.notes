---
note: 01KJBYT2C9Y03Q2BGZAVXP0HE8.md
title: 20221115 - OMA参数列表整理
indexed_at: 2026-02-27T10:19:53.386468+00:00
---

用户要求我分析一篇关于 OMA 参数列表的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是关于 OceanBase OMA (OceanBase Migration Assessment) 工具的参数整理笔记
2. 主要内容包括：
   - 各种工作模式（ANALYZE, ANALYZE_TOTAL, REPLAY, DUMP 等）
   - ANALYZE/ANALYZE_TOTAL 模式的工作流程分析（collect, convert, evaluate, performance evaluate, tuning, report）
   - 各阶段的详细参数说明

现在我来生成索引：

**标签**：需要 3-6 个关键词
- OceanBase
- OMA
- 迁移评估
- 参数配置
- SQL 分析
- 数据库迁移

**摘要**：2-3 句话
- 整理了 OceanBase OMA 工具的各工作模式及其参数配置
- 详细分析了 ANALYZE/ANALYZE_TOTAL 模式的六阶段工作流程（采集、转换、评估、性能评估、调优、报告）
- 列举了各阶段的核心参数选项

**关键概念**：3-5 个
- ANALYZE 模式：兼容性分析模式，用于评估 SQL 兼容性
- 工作流阶段：collect→convert→evaluate→performance evaluate→tuning→report 六个处理阶段
- evaluate-mode：SQL 规则评估模式，包括 ONLY_TARGET/SOURCE_TARGET/ONLY_SOURCE/ONLY_INSTANCE 等
- from-type：数据来源类型，支持 DB/TEXT/MYBATIS/WCR 等多种输入源


- 关联笔记方面，现有笔记库中缺少直接相关的 OMA 或迁移评估内容，虽然存在其他数据库相关笔记但关联度不高
## 标签
OceanBase, OMA, 迁移评估, 参数配置, SQL 分析, 数据库迁移

## 摘要
整理了 OceanBase OMA（迁移评估工具）的各工作模式及其参数配置。详细分析了 ANALYZE/ANALYZE_TOTAL 模式的六阶段工作流程：采集、转换、评估、性能评估、调优、报告。列举了各阶段的核心参数选项，如 from-type、evaluate-mode、collect 系列参数等。

## 关键概念
- ANALYZE 模式：兼容性分析模式，用于评估源库 SQL 与目标库的兼容性
- 工作流六阶段：collect→convert→evaluate→performance evaluate→tuning→report 的完整处理链路
- evaluate-mode：SQL 规则评估模式，包括 ONLY_TARGET/SOURCE_TARGET/ONLY_SOURCE/ONLY_INSTANCE 等选项
- from-type：数据来源类型，支持 DB/TEXT/MYBATIS/WCR/CODE 等多种输入源

## 关联笔记
- 01KJBYDACWHS9VPYASX7JF6Q61.md: 同为数据库迁移工具评估相关笔记（华夏银行迁移工具评估）
