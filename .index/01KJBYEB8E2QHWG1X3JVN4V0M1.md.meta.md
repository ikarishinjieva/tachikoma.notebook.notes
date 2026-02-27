---
note: 01KJBYEB8E2QHWG1X3JVN4V0M1.md
title: 20210819 - Memory Leak (and Growth) Flame Graphs 文章解析
indexed_at: 2026-02-27T10:09:00.736939+00:00
---

用户要求我分析一篇关于内存泄漏和火焰图的文章解析笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题是关于内存泄漏和内存增长的火焰图分析
2. 内容涵盖了多种内存诊断手段：memcheck、libtcmalloc
3. 追踪 Allocator 的 malloc/free
4. 追踪 brk 系统调用
5. 追踪 mmap 系统调用
6. 追踪缺页异常
7. libtcmalloc 的具体使用示例

从笔记列表中寻找关联：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 这个明显与内存相关
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 这个与内存指标相关

现在生成索引：

标签：内存诊断、火焰图、内存泄漏、系统调用、性能分析

摘要：介绍多种内存诊断手段，包括 memcheck、libtcmalloc 等工具。详细讲解了通过追踪 malloc/free、brk、mmap 系统调用和缺页异常来分析内存使用的方法。

关键概念：
- 火焰图：可视化内存分配和泄漏的图形工具
- brk 系统调用：用于改变数据段大小，追踪虚拟内存扩展


- mmap 系统调用：分配大内存或由 Allocator 调用
- 缺页异常：追踪物理内存的使用情况
- RSS/虚拟内存：区分虚拟内存分配与实际物理内存占用

关联笔记：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同样探讨内存分析和去向追踪
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 提供内存指标的详细解释## 标签
内存诊断，火焰图，内存泄漏，系统调用，性能分析

## 摘要
介绍多种内存诊断手段，包括 memcheck、libtcmalloc 等工具。详细讲解通过追踪 malloc/free、brk、mmap 系统调用和缺页异常来分析内存使用的方法，并对比各方案的追踪粒度与代价。

## 关键概念
- 火焰图：可视化内存分配和泄漏的图形工具
- brk 系统调用：改变数据段大小，用于追踪虚拟内存扩展
- mmap 系统调用：分配大内存，由 Allocator 调用
- 缺页异常：追踪物理内存（RSS）的使用情况
- Allocator：操作虚拟内存而非 RSS 的内存分配器

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同样是内存分析主题，探讨内存使用情况
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 解释 VSS/RSS/USS/PSS 内存指标，与本文虚拟内存和 RSS 概念相关
