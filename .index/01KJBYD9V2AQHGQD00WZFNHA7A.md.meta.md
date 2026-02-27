---
note: 01KJBYD9V2AQHGQD00WZFNHA7A.md
title: 20210409 - iostat 与 sar -b 的区别
indexed_at: 2026-02-27T10:01:26.653738+00:00
---

用户要求我分析一篇关于 iostat 和 sar -b 区别的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20210409 - iostat 与 sar -b 的区别
2. 内容主要对比了两个工具的数据来源和实现方式：
   - sar: 从 /proc/diskstats 读取数据
   - iostat: 从 /sys/block/{dev}/stat 读取数据

3. 关联笔记分析：
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema的日常使用 - 都是性能监控相关
   - 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 涉及磁盘 I/O
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 系统性能分析

让我生成符合格式的索引。## 标签
iostat, sar, 磁盘 IO, 性能监控, sysstat, Linux

## 摘要
对比分析 iostat 和 sar -b 两个磁盘 IO 监控工具的数据来源差异。sar 从 /proc/diskstats 直接读取累加，iostat 从 /sys/block/{dev}/stat 获取更详细的统计信息。

## 关键概念
- /proc/diskstats: Linux 内核提供的磁盘统计信息接口，sar 的数据源
- /sys/block/{dev}/stat: sysfs 中的块设备统计文件，iostat 的数据源
- S_VALUE: sysstat 库计算差值的宏，用于计算两次采样间的速率

## 关联笔记
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿: 同属磁盘 IO 性能分析主题
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用: 同为系统性能监控工具研究
