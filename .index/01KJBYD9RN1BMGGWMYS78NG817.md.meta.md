---
note: 01KJBYD9RN1BMGGWMYS78NG817.md
title: 20210404 - 如何给LWP降速
indexed_at: 2026-03-05T07:16:40.250885+00:00
---

## 标签
LWP, 线程限速, SIGSTOP, cpulimit, pthread, 多线程

## 摘要
记录两种给 LWP 降速方案的失败尝试。cpulimit 使用 SIGSTOP/SIGCONT 会影响整个进程，pthread_kill 发送 SIGSTOP 同样会停止所有线程而非单个线程。

## 关键概念
- cpulimit: 通过 procfs 读取 cpu_time，使用 SIGSTOP/SIGCONT 控制进程 CPU 使用的工具
- SIGSTOP/SIGCONT: 进程控制信号，作用于整个进程而非单个线程
- pthread_kill: POSIX 线程信号发送函数，但信号行为作用于进程级别
- LWP: 轻量级进程 (Light Weight Process)，即内核线程

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 笔记中引用的 GDB 单线程调试方法配置
