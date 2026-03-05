---
note: 01KJBYD9V2AQHGQD00WZFNHA7A.md
title: 20210409 - iostat 与 sar -b 的区别
indexed_at: 2026-03-05T07:21:35.593530+00:00
---

## 标签
iostat, sar, 磁盘 IO, sysstat, 性能监控, /proc/diskstats

## 摘要
分析 iostat 和 sar -b 两个工具的数据来源差异。sar 直接从/proc/diskstats 读取累加数据，而 iostat 从/sys/block/{dev}/stat 读取。两者数据同源但读取方式和输出格式不同。

## 关键概念
- /proc/diskstats: Linux 内核提供的磁盘统计信息接口，sar 的数据来源
- /sys/block/{dev}/stat: sysfs 中每块设备的统计信息文件，iostat 的数据来源
- sysstat: 包含 sar 和 iostat 的系统性能监控工具包
- S_VALUE: sysstat 中计算两点采样间速率的宏

## 关联笔记
- 01KJBYXQEJ0X39WB95B5E7PMN3.md: 同属 sar 工具探索相关的性能分析笔记
- 01KJBYJTJXB03VRDH5EHQ3VQWY.md: 涉及 vmstat 等系统监控工具的使用
- 01KJBYXQHSCD3DVCRPTGQVN88W.md: 包含 vmstat 和 dstat 采集的系统性能分析
