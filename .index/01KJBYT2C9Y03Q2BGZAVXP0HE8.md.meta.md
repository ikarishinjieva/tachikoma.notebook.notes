---
note: 01KJBYT2C9Y03Q2BGZAVXP0HE8.md
title: 20221115 - OMA参数列表整理
indexed_at: 2026-03-05T08:14:48.753057+00:00
---

## 标签
OMA, OceanBase, 迁移评估, 参数配置, SQL 分析, 数据库迁移

## 摘要
整理 OceanBase OMA（迁移评估工具）的工作模式与各阶段参数配置。涵盖 ANALYZE/ANALYZE_TOTAL 模式的六阶段工作流（采集、转换、评估、性能评估、调优、报告）及详细参数说明。

## 关键概念
- ANALYZE 模式: 兼容性分析模式，对 SQL 进行规则评估和转换建议
- ANALYZE_TOTAL 模式: 整库评估模式，支持对象、SQL、画像、推荐等多种分析类型
- evaluate-mode: SQL 规则评估模式，包括 ONLY_SOURCE/ONLY_TARGET/SOURCE_TARGET/ONLY_INSTANCE/APPLICATION_CODE
- from-type: 数据来源类型，支持 DB/TEXT/MYBATIS/WCR/COLLECT 等多种输入方式
- collect 阶段参数: 控制 SQL 采集的参数，包括采集时长、间隔、过滤条件等

## 关联笔记
- 01KJBYSTE6KE2T030VC4M9TP90.md: 同系列 OMA 评估笔记，包含 OMA 命令示例和模式说明
