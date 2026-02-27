---
note: 01KJBYDP3X93594WV0FTRG0KGM.md
title: 20210705 - linux的CPU频率
indexed_at: 2026-02-27T10:06:13.507963+00:00
---

用户要求我分析一篇关于 Linux CPU 频率的笔记，并生成索引元数据。让我仔细分析笔记内容：

笔记内容主要涉及：
1. /proc/stat 中 CPU 使用率的单位（USER_HZ）
2. /proc/pid/stat 中 CPU 使用率的单位（clock tick/_SC_CLK_TCK）
3. 相关的 Linux 内核配置（CONFIG_HZ）

核心概念：
- /proc/stat
- /proc/pid/stat
- USER_HZ
- clock tick
- CONFIG_HZ

关联笔记分析：
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 20210513 - 百胜 cpu 高的探索 - 与 CPU 相关
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 系统性能相关
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - 性能监控相关
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 系统资源相关

最相关的应该是 CPU 相关的笔记。

让我生成索引：

标签：Linux, CPU, /proc, 性能监控, 内核

摘要：笔记记录了 Linux 系统中/proc/stat 和/proc/pid/stat 文件中 CPU 使用率的计量单位。说明了 USER_HZ 和 clock tick(_SC_CLK_TCK) 的区别，以及如何查看内核 HZ 配置。


关键概念包括 /proc/stat 和 /proc/pid/stat 这两个系统接口，它们分别以 USER_HZ 和 clock tick 为单位计量 CPU 时间，而 CONFIG_HZ 则定义了内核时钟频率。在关联笔记中，百胜 cpu 高的探索这篇与 CPU 性能分析直接相关。## 标签
Linux, CPU, /proc, 性能监控, 内核参数

## 摘要
笔记记录了 Linux 系统中 `/proc/stat` 和 `/proc/pid/stat` 文件中 CPU 使用率的计量单位差异。前者使用 USER_HZ 单位，后者使用 clock tick (_SC_CLK_TCK) 单位，并提供了查看内核 CONFIG_HZ 配置的方法。

## 关键概念
- /proc/stat: 系统级 CPU 统计文件，使用 USER_HZ 单位
- /proc/pid/stat: 进程级 CPU 统计文件，使用 clock tick 单位
- USER_HZ: Linux 系统 CPU 使用率的计量单位
- CONFIG_HZ: 内核时钟频率配置参数

## 关联笔记
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 同为 CPU 性能问题排查相关的笔记
