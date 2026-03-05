---
note: 01KJBYM9JN59CM9N2ZHKRYFQ2R.md
title: 20220819 - 如何理解perf event: node-load-misses
indexed_at: 2026-03-05T08:07:49.028222+00:00
---

## 摘要
分析 Linux 内核中 Skylake 架构的 hardware cache event 定义，通过源码 skl_hw_cache_event_ids 和 skl_hw_cache_extra_regs 解读 node-load-misses 事件。node-load 对应 L3_MISS_LOCAL_DRAM（本地读），node-load-misses 对应 L3_MISS_REMOTE（远程读）。

## 关键概念
- skl_hw_cache_event_ids: Skylake 架构的硬件事件 ID 定义表，位于 Linux 内核源码 arch/x86/events/intel/core.c
- L3_MISS_LOCAL_DRAM: 本地 DRAM 访问的 L3 miss 事件标志
- L3_MISS_REMOTE: 远程节点访问的 L3 miss 事件标志，对应 node-load-misses
- node-load: NUMA 架构中本地节点的内存加载事件
- node-load-misses: NUMA 架构中远程节点的内存加载 miss 事件

## 关联笔记
- 01KJBYJV26733NF0YPZ7ETDET7.md: 同属 perf event 性能分析主题，包含 TMA 方法和 Intel VTune 指标解释
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 涉及 perf events 详解和 NUMA 内存访问性能分析，包含 node-load-misses 的测定方法
