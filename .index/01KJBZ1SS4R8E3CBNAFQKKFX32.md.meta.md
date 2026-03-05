---
note: 01KJBZ1SS4R8E3CBNAFQKKFX32.md
title: 20230714 - <Clickhouse性能之巅>
indexed_at: 2026-03-05T08:52:44.048380+00:00
---

## 标签
Clickhouse, 维度建模, 数据仓库, ORC, Parquet, 列式存储

## 摘要
《Clickhouse 性能之巅》读书笔记，涵盖数仓维度建模的 ODS/DW/ADS 三层架构及传统数仓缺陷。重点记录变长数据的三种处理方式，以及 ORC 与 Parquet 在不同存储场景下元数据位置的设计差异。

## 关键概念
- 维度建模: ODS 层、DW 层、ADS 层三层数据仓库架构
- ORC: 条带定长与 HDFS 块对齐，元数据放文件末尾避免破坏对齐优化
- Parquet: 非 HDFS 场景下字节对齐无收益，元数据可放文件开头
- 变长数据处理: 固定分隔符、长度数组、偏移量文件三种方式

## 关联笔记
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 同属 Clickhouse 技术研究，探讨 Join 实现原理
- 01KJBYDD58AJTW5NKT3GET7B89.md: Clickhouse 系统学习笔记，涵盖执行计划与知识框架
