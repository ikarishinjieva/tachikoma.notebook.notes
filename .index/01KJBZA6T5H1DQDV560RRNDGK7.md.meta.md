---
note: 01KJBZA6T5H1DQDV560RRNDGK7.md
title: 20240619 - ChatDBA: 如何判断参考材料和问题的相关性 [3]
indexed_at: 2026-03-05T10:13:29.305729+00:00
---

## 摘要
分析 ChatDBA 如何判断参考材料与用户问题的相关性，以虚构的"datetime is overflow"错误为例，评估三篇 MySQL 工单（load data 导入问题、timestamp 时区转换 CPU 高、从库故障）的参考价值。展示了从参考文档中提取有用信息并适配到具体问题的推理过程。

## 关键概念
- RAG 相关性判断: 评估检索到的参考文档与用户问题的匹配程度和适用性
- datetime overflow: MySQL 日期时间值超出有效范围（1000-01-01 到 9999-12-31）导致的错误
- load data: MySQL 导入 CSV 文件时因格式不匹配（行终止符、引号、日期转换函数）导致数据截断
- timestamp 时区转换: time_zone 设为 SYSTEM 时调用 OS API 进行时区转换导致 CPU %sys 升高

## 关联笔记
- 01KJBZZYFRDE3MNYM2SP16GZ3T.md: ChatDBA 代码仓库链接，与本笔记同属 ChatDBA 项目
- 01KJBYFD3950T3XNVP5R4BXYZV.md: 农行 load data 引发锁的分析，与本笔记中 load data 导入问题相关
