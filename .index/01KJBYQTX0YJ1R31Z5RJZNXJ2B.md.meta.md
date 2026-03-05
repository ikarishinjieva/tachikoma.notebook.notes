---
note: 01KJBYQTX0YJ1R31Z5RJZNXJ2B.md
title: 20221024 - OB 迁移评估 OMA
indexed_at: 2026-03-05T08:11:58.415324+00:00
---

## 摘要
记录 OceanBase 迁移评估工具 (OMA) 的评估阶段矩阵，涵盖 Oracle、MySQL、PostgreSQL、TiDB、DB2 LUW 等源数据库的对象兼容性和 SQL 语句兼容性评估维度。包含待确认问题：OMA 导出原理及 PORTRAIT 用户画像功能的具体含义。

## 关键概念
- OMA (OceanBase Migration Assessment): OceanBase 官方迁移评估工具，用于分析源数据库到 OceanBase 的兼容性
- 对象兼容性: 评估数据库对象（表、索引、视图等）从源库到目标库的迁移兼容程度
- SQL 语句兼容性: 评估 SQL 语法和语义在源库和目标库之间的兼容性
- PORTRAIT (用户画像): OMA 的数据库画像分析功能，用于采集数据库运行特征和负载信息
- 性能评估: 对源数据库进行性能基准测试，评估迁移后的性能表现

## 关联笔记
- 01KJBYSTE6KE2T030VC4M9TP90.md: 同系列 OMA 评估笔记，记录了 OMA 工具的工作流程、命令示例及模式详解
- 01KJBYT2C9Y03Q2BGZAVXP0HE8.md: OMA 参数列表整理，详细解释了各评估模式和命令行参数
