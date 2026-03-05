---
note: 01KJBYEGN81KQ8NPHZSF7TPQH2.md
title: 20210825 - Linux smaps的相关文档
indexed_at: 2026-03-05T07:42:11.592211+00:00
---

## 标签
Linux, smaps, glibc malloc, vdso, arena, heap

## 摘要
整理 Linux smaps 相关文档，涵盖 vdso/vsyscall 机制说明及 glibc malloc 的 arena/heap 内存分配模型。解释 64 位系统 arena 数量上限（8×核数）及 thread arena 与 heap 的层级关系。

## 关键概念
- vdso: virtual dynamic shared object，用于无需上下文切换的快速系统调用
- arena: glibc malloc 的内存管理区域，64 位系统上限为 8×CPU 核数
- heap: 属于 arena 的内存块，一个 arena 可包含多个 heap
- brk/mmap: glibc malloc 获取内存的两种 syscall，大内存请求使用 mmap

## 关联笔记
- 01KJBYED7JBE2YBE7Y4YY213H6.md: 引用本笔记分析 smaps 中的 heap 区域
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 解释 smaps 中的 VSS/RSS/USS/PSS 内存指标
- 01KJBYF3A6NVY01BBK3M77MSBX.md: 包含 vdso 内存映射区域分析
