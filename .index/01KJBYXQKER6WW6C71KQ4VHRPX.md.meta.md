---
note: 01KJBYXQKER6WW6C71KQ4VHRPX.md
title: 20220818 - 百胜slave延迟, 重新整理所有线索
indexed_at: 2026-03-05T08:24:00.330782+00:00
---

## 摘要
记录百胜 MySQL slave 延迟问题的诊断过程，发现 Skylake 架构下 ut_delay 较长、空闲连接过多、Cstate 设置等因素与延迟相关。通过 perf 分析 IPC、dcycles 等指标，验证禁用 Intel Idle Driver 和移除空闲连接可消除延迟。

## 关键概念
- ut_delay: Skylake 架构下 MySQL 自旋锁等待时间较长的现象
- Intel Idle Driver: 控制 CPU Cstate 的内核驱动，禁用后延迟不再发生
- NUMA Interleave: InnoDB 的 NUMA 内存交错分配策略，影响内存访问效率
- dcycles/cycles: CPU 停顿周期占比，有延迟时约 10%，正常约 5%
- THP (Transparent Huge Pages): 透明大页，可能影响数据库页表性能

## 关联笔记
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 同一问题的内存页表诊断记录
- 01KJBYKK56GGEG65TSY8WHHZXJ.md: 同一问题的上下文切换诊断记录
- 01KJBYJTJXB03VRDH5EHQ3VQWY.md: NUMA balancing 实验，为本次分析提供基础
