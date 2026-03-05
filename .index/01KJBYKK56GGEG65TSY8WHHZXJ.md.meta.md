---
note: 01KJBYKK56GGEG65TSY8WHHZXJ.md
title: 20220818 - 百胜slave延迟, 关于上下文切换的诊断
indexed_at: 2026-03-05T08:07:26.570921+00:00
---

## 摘要
通过 perf 诊断工具分析 MySQL 主从架构中大量空闲连接（40000 个，每秒 COM_PING）对 master 性能的影响。建立空闲连接后 sysbench TPS 从 33000 降至 14000 左右，context-switches、cpu-migrations、page-faults 等系统指标显著变化，最终定位到 COM_PING 导致的 CPU 压力问题。

## 关键概念
- context-switches: 上下文切换次数，反映线程调度频率和系统开销
- COM_PING: MySQL 协议中用于保活连接的命令，空闲连接定期发送以维持连接状态
- perf stat: Linux 性能分析工具，用于采集硬件事件计数器数据
- cpu-migrations: CPU 迁移次数，反映线程在不同 CPU 核心间切换的频率
- page-faults: 缺页中断次数，反映内存访问缺页情况

## 关联笔记
- 01KJBYKHHWQJ1G8B4FCQSJSMEX.md: 本笔记中引用的 golang 并发连接测试代码
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 同系列百胜 slave 延迟诊断，涉及内存页表和空闲连接分析
- 01KJBYJTJXB03VRDH5EHQ3VQWY.md: NUMA balancing 相关诊断，涉及 page fault 和 perf 分析方法
