---
note: 01KJBYKK56GGEG65TSY8WHHZXJ.md
title: 20220818 - 百胜slave延迟, 关于上下文切换的诊断
indexed_at: 2026-02-27T10:18:09.839571+00:00
---

用户要求我分析这篇关于 MySQL 性能诊断的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 标题：20220818 - 百胜 slave 延迟，关于上下文切换的诊断
2. 内容主要讲述：
   - 测试环境中 1 主 1 从架构
   - 向 slave 建立 40000 个空闲连接（每秒 COM_PING）
   - 使用 sysbench 向 master 发起压力
   - 发现建立空闲连接后 master 性能下降
   - 使用 perf 工具进行性能分析，关注 context-switches（上下文切换）

核心主题：
- MySQL 性能问题诊断
- 上下文切换（context-switches）
- 主从复制/slave 延迟
- perf 性能分析工具
- 大量空闲连接对性能的影响

关联笔记分析：
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 20210513 - 百胜 cpu 高的探索 - 同是百胜项目的 CPU 问题
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - 性能分析相关
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 20210407 - 同样的压力，不同机器 perf 呈现的热点不同 - perf 性能分析相关

让我生成索引元数据。
## 标签
MySQL, 上下文切换, 性能诊断, 主从复制, perf, 连接数

## 摘要
在 1 主 1 从架构中，向 slave 建立 40000 个空闲连接后，master 的 sysbench 压测性能显著下降（tps 从 35000 降至约 14000）。通过 perf 对比分析发现，建立连接后 master 和 slave 的 context-switches 明显增加，定位上下文切换是性能瓶颈。

## 关键概念
- context-switches: 上下文切换次数，反映线程调度开销，过高会导致 CPU 效率下降
- COM_PING: MySQL 空闲连接的心跳命令，大量并发时会增加服务器负载
- perf: Linux 性能分析工具，用于统计 CPU、分支预测、上下文切换等指标
- 主从架构: 1 主 1 从的 MySQL 复制拓扑，slave 负载会影响整体性能

## 关联笔记
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 同是百胜项目的 CPU 性能问题探索
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 同样涉及 perf 工具分析不同机器性能热点差异
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 涉及 performance_schema 性能分析工具的使用
