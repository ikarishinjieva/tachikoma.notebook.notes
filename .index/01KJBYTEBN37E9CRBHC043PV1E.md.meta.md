---
note: 01KJBYTEBN37E9CRBHC043PV1E.md
title: 20221116 - MySQL在arm/x86上, 性能不同
indexed_at: 2026-03-05T08:15:26.721344+00:00
---

## 摘要
记录 MySQL 8.0.24 在 ARM 架构（麒麟 V10）上创建空间索引性能异常（41s vs x86 的 4s）的问题排查过程。通过火焰图分析定位到 libm 额外消耗，最终发现是 BOOST 库版本问题（8.0.29 从 1.67.0 升级到 1.77.0 后修复）。

## 关键概念
- 空间索引 (Spatial Index): MySQL 用于加速地理空间数据查询的索引类型
- 火焰图 (Flame Graph): 可视化性能分析工具，展示 CPU 时间在各函数中的分布
- SRID: 空间参考系统标识符，用于定义坐标系统的标准
- BOOST 库: C++ 跨平台库，MySQL 依赖其进行几何计算

## 关联笔记
- 01KJBYVPR3SFP3J23BJMBGW4NV.md: 同环境（麒麟 V10）下 MySQL 8.0 编译问题记录
- 01KJBYFQ942KXQ4NR803SNC0T2.md: MySQL 8.0 与 5.7 的 insert 性能差异分析
