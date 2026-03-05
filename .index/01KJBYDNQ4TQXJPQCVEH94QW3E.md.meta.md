---
note: 01KJBYDNQ4TQXJPQCVEH94QW3E.md
title: 20210628 - VSS/RSS/USS/PSS 解释
indexed_at: 2026-03-05T07:33:47.735149+00:00
---

## 标签
Linux 内存，进程内存测量，VSS/RSS/USS/PSS, procrank, 嵌入式 Linux, 内存分析

## 摘要
介绍 Linux 进程内存测量的四种指标（VSS/RSS/USS/PSS）及其计算方式和适用场景。解释了 procrank 工具的原理和用途，相比 smem 更适合嵌入式 Linux 环境。

## 关键概念
- VSS (Virtual Set Size): 进程映射的总虚拟内存，包含已分配但未使用的内存
- RSS (Resident Set Size): 进程占用的物理内存，但未扣除共享页面导致重复计算
- USS (Unique Set Size): 进程独占的私有内存， fork 新进程时的内存代价
- PSS (Proportional Set Size): 按比例分摊共享页面后的内存占用，所有进程 PSS 之和等于总内存使用量
- procrank: 基于/proc/[PID]/smaps 的 C 语言命令行工具，无需 Python 运行时

## 关联笔记
- 01KJBYEB8E2QHWG1X3JVN4V0M1.md: 同样讨论内存诊断，涉及 RSS 和虚拟内存概念
- 01KJBYEJ6EH6E3EXVZ6D3M5TWH.md: 涉及/proc/*/smaps 的使用，与 procrank 数据源相同
- 01KJBYDP3X93594WV0FTRG0KGM.md: 关于/proc 文件系统的说明，是理解 procrank 的基础
