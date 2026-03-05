---
note: 01KJBYJV26733NF0YPZ7ETDET7.md
title: 20220623 - Top-down Microarchitecture Analysis 尝试
indexed_at: 2026-03-05T08:01:33.098060+00:00
---

## 摘要
记录使用 Intel VTune 进行 Top-down Microarchitecture Analysis (TMA) 的尝试，分析 Memory Bound 指标计算方式。通过 pmu-tools 对比同 NUMA 与跨 NUMA 访问 MySQL 的性能差异，发现跨 NUMA 时 Memory_Bound 升高约 15%。

## 关键概念
- TMA (Top-down Microarchitecture Analysis): 自顶向下的 CPU 架构性能瓶颈分析方法，将性能问题分为 Frontend Bound、Backend Bound、Bad Speculation、Retiring 四类
- Memory Bound: TMA 指标之一，表示内存子系统成为性能瓶颈的周期占比
- Pipeline Slots: CPU 微架构中指令执行的槽位单位，用于量化各性能瓶颈的占比
- perf event: Linux 性能监控硬件事件，如 CPU_CLK_UNHALTED、MEM_LOAD_RETIRED 等

## 关联笔记
- 01KJBYJTJXB03VRDH5EHQ3VQWY.md: 同一实验系列的 NUMA balancing 配置与测试笔记，本笔记实验 1 沿用了该笔记的测试场景
