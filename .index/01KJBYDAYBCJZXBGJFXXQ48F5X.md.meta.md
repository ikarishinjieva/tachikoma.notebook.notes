---
note: 01KJBYDAYBCJZXBGJFXXQ48F5X.md
title: 20210415 - Clickhouse MaterializedMySQL 学习
indexed_at: 2026-03-05T07:25:30.673818+00:00
---

## 摘要
记录 ClickHouse MaterializeMySQL 引擎的学习笔记，涵盖虚拟列机制、DDL/DML 转换规则、binlog 同步流程及代码入口。包含复制进度查看、数据库 UUID 查找和故障恢复方法。

## 关键概念
- 虚拟列: _version 为事务计数器，_sign 为删除标志（-1 表示删除）
- binlog 同步: MySQL 变更通过 binlog event 转换为 ClickHouse INSERT 操作
- optimize_on_insert: 插入前进行数据合并的优化机制
- Buffer 刷新: 基于时间间隔、行数或字节数阈值触发 buffer flush

## 关联笔记
- 01KJBYE4H01Z06Q70JXCFR0THQ.md: 同为 MaterializeMySQL 引擎的实战问题记录，包含建库配置和错误堆栈
