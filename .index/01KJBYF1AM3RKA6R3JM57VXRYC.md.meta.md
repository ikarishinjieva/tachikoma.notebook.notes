---
note: 01KJBYF1AM3RKA6R3JM57VXRYC.md
title: 20211001 - 为什么tcmalloc会慢
indexed_at: 2026-03-05T07:46:14.622444+00:00
---

## 摘要
调查 tcmalloc 导致性能下降的原因，发现 HeapProfileTable::GetCallerStackTrace 调用 _Unwind_Backtrace 消耗过大。对比 jemalloc 发现其通过 lg_prof_sample 采样间隔（默认 512 KiB）减少调用次数，从而获得更好性能。

## 关键概念
- tcmalloc: Google 开发的线程缓存内存分配器
- jemalloc: FreeBSD 开发的高性能内存分配器
- HeapProfileTable::GetCallerStackTrace: tcmalloc 中获取调用堆栈的函数
- _Unwind_Backtrace: 底层堆栈追踪函数，性能开销大
- lg_prof_sample: jemalloc 的采样间隔参数，控制堆栈追踪频率

## 关联笔记
无
