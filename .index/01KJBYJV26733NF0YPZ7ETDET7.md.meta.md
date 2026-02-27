---
note: 01KJBYJV26733NF0YPZ7ETDET7.md
title: 20220623 - Top-down Microarchitecture Analysis 尝试
indexed_at: 2026-02-27T10:17:12.076790+00:00
---

用户要求我分析这篇关于 Intel Vtune 性能分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：尝试使用 Intel Vtune 进行 Top-down Microarchitecture Analysis
2. 主要内容：
   - 尝试安装和使用 Intel Vtune
   - 远程调试 Linux 程序未成功
   - 研究 Vtune 如何计算 Memory bound
   - 包含 bottom-up 分析数据和 Event count 数据

3. 关键概念：
   - Top-down Microarchitecture Analysis
   - Memory Bound
   - Pipeline Slots
   - Performance Events
   - Vtune

4. 关联笔记分析：
   - 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 20210407 - 同样的压力，不同机器 perf 呈现的热点不同 - 都与性能分析相关
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 都与内存分析相关
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210404 - performance_schema 的日常使用 - 都与性能监控相关

现在生成输出。## 标签
性能分析, Intel Vtune, Microarchitecture, Memory Bound, Top-down Analysis, Perf Events

## 摘要
记录尝试使用 Intel Vtune 进行 Top-down Microarchitecture Analysis 的过程，包括安装使用和远程调试遇到的问题。分析了 Vtune 如何计算 Memory Bound 指标，包含具体的 bottom-up 分析数据和硬件事件计数。

## 关键概念
- Top-down Microarchitecture Analysis: 通过 Pipeline Slots 分类（Retiring/Front-End Bound/Back-End Bound/Bad Speculation）定位性能瓶颈的分析方法
- Memory Bound: 后端瓶颈中由内存访问延迟或带宽限制导致的性能问题占比
- Pipeline Slots: 衡量 CPU 流水线利用率的单位，用于量化微架构效率
- Perf Events: CPU 硬件事件计数器，如 CYCLE_ACTIVITY.STALLS_L3_MISS、MEM_LOAD_RETIRED 等

## 关联笔记
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 都涉及性能分析工具（perf）和热点定位问题
- 01KJBYDAF12RN18BECZG1QSTEC.md: 都关注内存相关的性能问题分析
