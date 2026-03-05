---
note: 01KJBZ418EW7DDYY2KFX8YWYNF.md
title: 20231121 - async-profiler在容器中抓取java的火焰图
indexed_at: 2026-03-05T09:19:09.151492+00:00
---

## 标签
async-profiler, Java, 火焰图, 容器, profiling, fdtransfer

## 摘要
记录在容器中使用 async-profiler 抓取 Java 进程火焰图的命令流程。通过 profiler.sh 的 start/status/stop 三步操作完成性能采样。

## 关键概念
- async-profiler: Java 性能分析工具，支持 CPU/内存采样生成火焰图
- 火焰图 (flamegraph): 可视化性能分析结果的图形，展示函数调用栈的耗时分布
- fdtransfer: 文件描述符传递选项，用于容器内 profiling 时传递 perf 事件数据

## 关联笔记
- 01KJBZ45XPJ5T7FYJVGSDF5EKH.md: 同样使用 async-profiler 的 profiler.sh 命令抓取火焰图
