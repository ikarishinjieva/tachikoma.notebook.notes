---
note: 01KJBYEB8E2QHWG1X3JVN4V0M1.md
title: 20210819 - Memory Leak (and Growth) Flame Graphs 文章解析
indexed_at: 2026-03-05T07:40:36.189758+00:00
---

## 摘要
解析 Brendan Gregg 关于内存泄漏和内存增长火焰图的文章，介绍多种内存诊断手段。涵盖从应用层到内核层的追踪方法，包括 Allocator 追踪、系统调用追踪（brk/mmap）和缺页异常追踪。

## 关键概念
- Flame Graph: 可视化性能分析工具，用于定位内存泄漏和增长的调用路径
- malloc/free: 内存分配器接口，操作虚拟内存而非 RSS
- brk 系统调用: 改变数据段大小，用于追踪虚拟内存扩大
- mmap 系统调用: 分配大内存，追踪代价低但粒度较粗
- 缺页异常: 追踪物理内存使用，反映实际内存消耗

## 关联笔记
- 01KJC02BRZ0WYFF71HAK5SAJGA.md: 包含使用 strace、bpftrace 追踪 mmap 和 brk 系统调用的实战案例
