---
note: 01KJBYEGN81KQ8NPHZSF7TPQH2.md
title: 20210825 - Linux smaps的相关文档
indexed_at: 2026-02-27T10:09:42.133423+00:00
---

用户要求我分析一篇关于 Linux smaps 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

笔记主要内容包括：
1. 文档1：关于 Linux 内存布局的文档，涉及 vdso、vsyscall、entry point、brk 和 mmap 的关系、glibc malloc 的工作原理
2. 文档2：关于 glibc malloc 的详细分析，包括 arena、heap、chunk 的概念

核心主题：Linux 内存管理、glibc malloc、smaps 分析

标签应该包括：Linux、内存管理、glibc、malloc、smaps 等

摘要：这篇笔记记录了 Linux smaps 相关的文档，包括虚拟动态共享对象 (vdso) 的说明，以及 glibc malloc 如何通过 brk 和 mmap 系统调用获取内存，还有 arena/heap/chunk 的层级关系。

关键概念：
- vdso: 虚拟动态共享对象，用于系统调用切换到内核态
- brk/mmap: glibc malloc 获取内存的两种系统调用方式
- arena: glibc malloc 中的内存区域，64 位系统上限为 8*核数
- heap: 线程 arena 可包含多个 heap
- chunk: heap 由多个 chunk 组成

关联笔记：
查看其他笔记列表，找相关的：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 这个和内存相关，可能有关联


- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 这个和 smaps 直接相关，因为 smaps 中会显示这些内存指标

最相关的应该是 VSS/RSS/USS/PSS 那篇，因为 smaps 是这些指标的来源。内存去哪儿了那篇也可能相关。## 标签
Linux, 内存管理, glibc, malloc, smaps, 内存分配

## 摘要
这篇笔记整理了 Linux smaps 相关的技术文档，涵盖 vdso 机制和 glibc malloc 的内存分配原理。重点说明了 brk/mmap 两种系统调用的使用场景，以及 arena/heap/chunk 的层级关系。

## 关键概念
- vdso: 虚拟动态共享对象，用于系统调用无需用户态和内核态切换
- brk/mmap: glibc malloc 获取内存的两种系统调用，大内存请求使用 mmap
- arena: glibc 内存区域，64 位系统上限为 8*核数，每个线程一个
- heap: 一个 thread arena 可包含多个 heap，只属于一个 arena
- chunk: heap 的基本组成单元，由多个 chunk 构成

## 关联笔记
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 同属内存分析主题，VSS/RSS/USS/PSS 是 smaps 中的关键指标
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同为内存分析相关笔记，探讨内存使用情况
