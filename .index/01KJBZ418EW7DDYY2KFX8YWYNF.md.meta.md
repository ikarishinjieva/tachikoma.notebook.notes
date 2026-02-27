---
note: 01KJBZ418EW7DDYY2KFX8YWYNF.md
title: 20231121 - async-profiler在容器中抓取java的火焰图
indexed_at: 2026-02-27T10:39:05.500229+00:00
---

用户要求我分析一篇关于 async-profiler 在容器中抓取 Java 火焰图的笔记，并生成索引元数据。让我分析笔记内容：

1. 笔记标题：20231121 - async-profiler 在容器中抓取 java 的火焰图
2. 笔记内容：展示了 async-profiler 的三个命令（start、status、stop），用于对进程 44853 进行性能分析，使用 fdtransfer 选项（用于容器环境），输出火焰图

需要生成：
- 标签：3-6 个关键词
- 摘要：2-3 句话
- 关键概念：3-5 个核心概念
- 关联笔记：从给定的笔记列表中找 0-3 篇相关的

看关联笔记，这些主要是关于 MySQL、Clickhouse、性能分析等主题的。与当前笔记（Java 性能分析、火焰图、async-profiler）最相关的可能是一些性能分析相关的笔记，比如：
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 20210407 - 同样的压力，不同机器 perf 呈现的热点不同（涉及性能分析）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（性能相关）

但这些主要是 MySQL 相关的，与 Java/async-profiler 关联度不高。

不过从性能分析这个共同主题来看，perf 那篇笔记还是有一定的关联性。现在开始整理输出。
## 标签
async-profiler, Java, 火焰图，容器，性能分析，fdtransfer

## 摘要
记录在容器环境中使用 async-profiler 抓取 Java 应用火焰图的命令流程。通过 fdtransfer 选项解决容器内 perf_event 权限问题，完成从启动、状态检查到停止导出火焰图的全过程。

## 关键概念
- async-profiler: Java 应用性能分析工具，支持生成火焰图
- fdtransfer: 容器环境下传递文件描述符以获取 perf 事件权限的机制
- 火焰图：可视化展示程序调用栈热点的性能分析图表

## 关联笔记
- 01KJBYD9SKSCBFSFV4QPZYZ5RX.md: 同样涉及性能分析工具（perf）和热点定位场景
