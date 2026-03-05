---
note: 01KJBYXQPQ1SX1N9ZQFRYPFKGG.md
title: 20220915 - 光大clickhouse诊断
indexed_at: 2026-03-05T08:26:24.896975+00:00
---

## 摘要
记录光大银行 ClickHouse 入库压力大导致 Too many parts(600) 报错问题。日志显示 ReplicatedMergeTree 表 CK_BLACK_PHONE_LOCAL 存在分区合并异常，多个 Part Check 线程持续检测缺失分区但合并未正常执行。

## 关键概念
- Too many parts: ClickHouse 当未合并的数据分区数超过阈值时触发的错误，影响写入性能
- ReplicatedMergeTree: ClickHouse 支持副本复制的 MergeTree 表引擎，需协调多节点分区状态
- MergerMutator: 负责后台分区合并的组件，合并失败会导致分区堆积
- Part Check Thread: 检查分区完整性的后台线程，发现缺失分区时尝试等待合并修复

## 关联笔记
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: 同一工单 BEIJ-2780 的后续诊断，记录 AST is too big 报错及复制表阻塞问题
- 01KJBYVTTVNYTSRBX5JBT2MJKP.md: 光大 AST is too big 报错的后续处理，涉及参数调优尝试
- 01KJBYWMG762P36NB4CPT3P64J.md: 光大 CK DELETE 数据失败问题，同一客户的 ClickHouse 故障记录
