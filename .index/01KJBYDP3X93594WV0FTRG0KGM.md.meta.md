---
note: 01KJBYDP3X93594WV0FTRG0KGM.md
title: 20210705 - linux的CPU频率
indexed_at: 2026-03-05T07:34:39.311421+00:00
---

## 摘要
记录 Linux 系统中 CPU 使用率时间单位的两种标准：/proc/stat 使用 USER_HZ 单位，/proc/pid/stat 使用 clock tick(_SC_CLK_TCK) 单位。提供查看 CONFIG_HZ 配置的方法，用于理解 CPU 时间统计的基准频率。

## 关键概念
- USER_HZ: /proc/stat 中 CPU 使用率的计量单位
- _SC_CLK_TCK: 系统每秒的 clock tick 数，用于/proc/pid/stat
- CONFIG_HZ: 内核时钟频率配置，决定 clock tick 的基准值
- /proc/stat: 提供系统级 CPU 使用统计信息的虚拟文件
- /proc/pid/stat: 提供进程级 CPU 使用统计信息的虚拟文件

## 关联笔记
- 01KJBYXQHSCD3DVCRPTGQVN88W.md: 涉及/proc/interrupts 和 turbostat 工具，同属 CPU 性能监控相关
- 01KJBYJTJXB03VRDH5EHQ3VQWY.md: 涉及/proc/vmstat 和 NUMA 配置，同属/proc 文件系统性能调优
- 01KJBYD9V2AQHGQD00WZFNHA7A.md: 涉及/proc/diskstats，同属/proc 文件系统监控指标
