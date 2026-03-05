---
note: 01KJBYJTJXB03VRDH5EHQ3VQWY.md
title: 20220612 - numa blancing
indexed_at: 2026-03-05T08:01:08.628100+00:00
---

## 标签
NUMA, 内存管理，Page Fault, Linux 内核，性能优化，iowait

## 摘要
记录 NUMA balancing 的三种工作模式：周期性扫描标记内存、触发 NUMA Hinting Fault、数据迁移到任务所在节点。整理 /proc/vmstat 和 /sys/devices/system/node/node*/numastat 中的关键统计指标及其含义。

## 关键概念
- NUMA Hinting Fault: 访问被标记的内存时触发的特殊 page fault，用于驱动内存迁移
- numa_pages_migrated: 因位置不当而迁移的页面数，是 NUMA balancing 开销的主要来源
- numa_pte_updates: 被标记为触发 NUMA hinting fault 的基页数量
- local_node: 进程在本节点 CPU 运行并从本节点分配内存的成功次数
- numa_foreign: 进程想在本节点分配内存但实际从其他节点分配的次数

## 关联笔记
- 01KJBYXQHSCD3DVCRPTGQVN88W.md: 深入分析 NUMA balancing 通过 task_numa_work 和 change_prot_numa 触发 TLB flush 导致 CAL 中断升高的问题
- 01KJBYJV26733NF0YPZ7ETDET7.md: 使用 TMA 方法测试跨 NUMA 访问时 Memory_Bound 指标升高 15% 的实验记录
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 讨论 NUMA 与 THP 配合导致的性能问题及 page fault handler 耗时分析
