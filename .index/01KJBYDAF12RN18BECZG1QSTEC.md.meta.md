---
note: 01KJBYDAF12RN18BECZG1QSTEC.md
title: 20210519 - 内存去哪儿了
indexed_at: 2026-02-27T10:02:03.715679+00:00
---

用户要求我分析这篇关于 Linux 内存分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：Linux 内存分析，主要讲解 /proc/meminfo 的各个字段含义
2. 核心内容：
   - /proc/meminfo 各字段解释（MemTotal, MemFree, Buffers, Cached, Slab等）
   - 内存分配分类（页分配、slab 分配、vmalloc、用户态分配）
   - 各种内存概念（Bounce、Hugepage、AnonHugePages 等）

3. 标签应该是：Linux, 内存分析，/proc/meminfo, 内核，内存分配

4. 摘要：这篇笔记详细解释了 Linux /proc/meminfo 文件中各字段的含义，并介绍了内核内存分配的分类机制（页分配、slab、vmalloc 等）。

5. 关键概念：
   - /proc/meminfo: Linux 内核内存信息接口
   - Slab 分配器：内核对象缓存分配机制
   - 匿名页：不包含文件映射的内存页
   - 大页 (Hugepage)：大于标准 4KB 的内存页

6. 关联笔记：查看其他笔记列表，找相关的
   - 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 这个与内存相关
   - 其他笔记主要是关于 MySQL、Clickhouse、gdb 等，与内存分析关联不大

现在生成输出。## 标签
Linux, 内存分析，/proc/meminfo, 内核内存，内存分配，Slab

## 摘要
这篇笔记详细解释了 Linux `/proc/meminfo` 文件中各字段的含义，包括 MemTotal、Buffers、Cached、Slab 等关键指标。同时介绍了内核内存分配的四种分类机制（页分配、slab 分配、vmalloc、用户态分配）以及大页、Bounce buffer 等特殊内存概念。

## 关键概念
- /proc/meminfo: Linux 内核提供的内存状态信息接口，展示系统内存使用情况
- Slab 分配器: 内核基于页分配机制建立的对象缓存分配系统，分为可回收 (SReclaimable) 和不可回收 (SUnreclaim) 部分
- 匿名页 (AnonPages): 不包含文件映射的内存页，包括进程堆栈和 THP 页面
- 大页 (Hugepage): 大于标准 4KB 的内存页，可减少 TLB 缺失，分为传统大页和匿名大页
- Bounce Buffer: 当 DMA 设备只能访问低端内存时，内核在低端内存分配的临时中转缓冲区

## 关联笔记
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: VSS/RSS/USS/PSS 解释，同为 Linux 进程内存相关的概念解析
