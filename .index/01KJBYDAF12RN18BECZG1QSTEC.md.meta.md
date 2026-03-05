---
note: 01KJBYDAF12RN18BECZG1QSTEC.md
title: 20210519 - 内存去哪儿了
indexed_at: 2026-03-05T07:23:57.663431+00:00
---

## 摘要
详解 Linux /proc/meminfo 各字段含义，包括匿名页、缓存页、slab、页表等内存统计项。系统介绍页分配、slab 分配、vmalloc、用户态分配四种内存分配机制及其特点。

## 关键概念
- /proc/meminfo: Linux 内核内存信息接口，提供系统内存使用情况的详细统计
- 页分配 (buddy system): 内核基于伙伴系统算法的物理页分配机制，通过 alloc_pages() 分配
- slab 分配: 在页分配基础上构建的对象缓存分配器，以字节为单位分配物理连续内存
- vmalloc: 内核虚拟地址空间分配机制，分配虚拟连续但物理可不连续的内存块
- 大页 (Hugepage): 使用大于标准 4KB 的内存页（如 2MB/1GB），减少 TLB miss 提升性能

## 关联笔记
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 内存页表性能诊断实战，分析 TLB miss 对数据库延迟的影响
- 01KJBYEJ6EH6E3EXVZ6D3M5TWH.md: THP 透明大页配置详解，包括 enabled/defrag/khugepaged 参数说明
- 01KJBYEK8DZVDASX1CPQPXC5DA.md: 通过 perf 和 sysbench 测试 THP 对 TLB 命中率的实际影响
