---
note: 01KJBYTEBN37E9CRBHC043PV1E.md
title: 20221116 - MySQL在arm/x86上, 性能不同
indexed_at: 2026-02-27T10:20:05.825743+00:00
---

用户要求我分析一篇关于 MySQL 在 ARM/x86 架构上性能差异的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 主要内容：MySQL 8.0.24 在 ARM 架构上创建空间索引很慢（41s vs 4s），升级到 8.0.31 后正常
2. 原因：BOOST 库升级（1.67.0→1.77.0）修复了 mbr_join_area 代码的问题
3. 涉及技术：MySQL、空间索引、ARM 架构、性能分析、火焰图、gdb、libm

关联笔记分析：
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: "同样的压力，不同机器 perf 呈现的热点不同" - 与性能分析相关
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: "performance_schema 的日常使用" - MySQL 性能相关
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 临时磁盘表相关 - 不太相关
- 其他笔记大多是 Clickhouse、gdb 等，关联度不高

最相关的是性能分析和 MySQL 相关的笔记。

现在生成元数据：

标签：MySQL, ARM 架构，性能优化，空间索引，BOOST 库，火焰图

摘要：记录 MySQL 8.0.24 在 ARM 架构上创建空间索引性能异常（41s vs x86 的 4s）的问题排查过程。
通过火焰图分析发现额外消耗于 libm，最终定位到 BOOST 库版本问题（1.67.0→1.77.0 升级后修复）。

关键概念：
- 空间索引 (Spatial Index): MySQL 用于地理空间数据的索引类型
- 火焰图 (Flame Graph): 性能分析工具，可视化展示 CPU 调用栈
- BOOST 库：C++ 库，MySQL 依赖其几何计算功能
- mbr_join_area: BOOST 中计算最小边界矩形相交面积的函数
- libm: 数学库，ARM 上某些数学运算性能差异的根源

关联笔记：
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 同样涉及不同机器性能热点差异的分析
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: MySQL performance_schema 性能监控相关## 标签
MySQL, ARM 架构，性能优化，空间索引，BOOST 库，火焰图

## 摘要
记录 MySQL 8.0.24 在 ARM 架构上创建空间索引性能异常（41s vs x86 的 4s）的问题排查过程。通过火焰图分析发现额外消耗于 libm，最终定位到 BOOST 库版本问题（8.0.29 从 1.67.0 升级到 1.77.0 后修复）。

## 关键概念
- 空间索引 (Spatial Index): MySQL 用于地理空间数据的索引类型，基于 R-Tree 实现
- 火焰图 (Flame Graph): 性能分析工具，可视化展示 CPU 调用栈热点
- BOOST 几何库: MySQL 依赖的 C++ 库，提供空间计算功能
- mbr_join_area: BOOST 中计算最小边界矩形相交面积的函数，问题根源
- libm: C 标准数学库，ARM 上某些数学运算存在性能差异

## 关联笔记
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 同样涉及不同机器 perf 呈现热点差异的性能分析
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: MySQL performance_schema 性能监控相关
