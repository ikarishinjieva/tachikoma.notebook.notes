---
note: 01KJBYXQEJ0X39WB95B5E7PMN3.md
title: 20220826 - 百胜slave延迟, sar探索
indexed_at: 2026-03-05T08:21:48.850670+00:00
---

## 标签
MySQL, 主从复制，延迟诊断，sar, pgpgout, 上下文切换

## 摘要
分析百胜 slave 延迟问题，通过 sar 工具探索系统性能指标与延迟的关联。发现 pgpgout 峰值、上下文切换、CPU 奇偶切换与延迟曲线高度重合，bwrtn/s 和 IO 指标趋势一致。

## 关键概念
- pgpgout/s: 每秒从内存页写出的数据量，峰值与延迟上升起点重合
- 上下文切换: 进程/线程在 CPU 间的切换，20 日/21 日出现异常波动
- CPU 奇偶切换: 偶数 CPU 与奇数 CPU 间的负载切换，与延迟起止时间重合
- await/svctm: IO 延迟指标，写入量增大时反而变低
- bwrtn/s: 块设备写入速率，与 pgpgout 趋势相同

## 关联笔记
- 01KJBYXQKER6WW6C71KQ4VHRPX.md: 同系列线索整理笔记，引用了内存页表诊断和上下文切换诊断
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 百胜 slave 延迟的内存页表相关诊断，与本笔记 pgpgout 分析相关
- 01KJBYKK56GGEG65TSY8WHHZXJ.md: 百胜 slave 延迟的上下文切换诊断，与本笔记上下文切换分析相关
