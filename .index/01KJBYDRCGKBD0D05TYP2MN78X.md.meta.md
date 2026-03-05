---
note: 01KJBYDRCGKBD0D05TYP2MN78X.md
title: 20210715 - 一个insert的p_s输出
indexed_at: 2026-03-05T07:37:30.392528+00:00
---

## 摘要
记录单机 INSERT 语句在 performance_schema 中的完整执行轨迹，包括 statement、stage、wait 三类事件表的查询结果。通过 events_statements_history_long、events_stages_history_long、events_waits_history_long 分析 INSERT 操作的执行阶段和等待事件。

## 关键概念
- events_statements_history_long: 记录 SQL 语句执行历史，包含执行时间、影响行数等统计信息
- events_stages_history_long: 记录 SQL 执行的各个阶段，如 opening tables、update、closing tables 等
- events_waits_history_long: 记录等待事件，包括 IO、锁、互斥量等底层资源等待

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 日常使用方法，包含 ps_setup 配置命令
- 01KJBYFD3950T3XNVP5R4BXYZV.md: 使用 performance_schema.data_locks 分析锁问题的案例
