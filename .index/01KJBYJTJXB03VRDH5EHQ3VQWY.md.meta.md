---
note: 01KJBYJTJXB03VRDH5EHQ3VQWY.md
title: 20220612 - numa blancing
indexed_at: 2026-02-27T10:17:02.861920+00:00
---

用户要求我分析一篇关于 NUMA balancing 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：NUMA balancing（NUMA 平衡）
2. 主要内容：
   - NUMA balancing 的工作模式（task scanner、page fault、内存迁移）
   - /proc/vmstat 中的 NUMA 相关统计值
   - /sys/devices/system/node/node*/numastat 的统计值
   - /proc/pid/numa_maps 的输出解释
   - 涉及 MySQL 和数据库相关的 NUMA 问题

3. 标签应该包括：NUMA、内存管理、性能优化、Linux 内核等

4. 关联笔记分析：
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 与内存相关
   - 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释 - 与内存相关
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 - 涉及 I/O 和内存
   - 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 涉及 I/O

这篇笔记主要讲 NUMA balancing 机制和相关的监控指标，与内存管理和性能优化相关。

关联笔记中，"内存去哪儿了"和"VSS/RSS/USS/PSS 解释"都涉及内存分析，但关联性不算特别强。我应该生成简洁的索引元数据。## 标签
NUMA, 内存管理, Linux 内核，性能优化，Page Fault

## 摘要
介绍 NUMA balancing 的工作机制，包括任务扫描器周期性标记内存、触发 NUMA Hinting Fault 及页面迁移的过程。整理了 `/proc/vmstat`、`/sys/devices/system/node/node*/numastat`、`/proc/pid/numa_maps` 等关键监控指标的说明。

## 关键概念
- NUMA Hinting Fault: 访问被标记的内存时触发的缺页中断，用于驱动页面迁移
- numa_pages_migrated: 因位置不当而迁移的页面数，是 NUMA balancing 主要开销来源
- local_node: 进程在本节点 CPU 运行并从本节点分配内存的次数

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（都涉及内存分析与监控）
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（都涉及内存指标解读）
