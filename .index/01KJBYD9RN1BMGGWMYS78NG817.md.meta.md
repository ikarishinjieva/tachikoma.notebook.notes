---
note: 01KJBYD9RN1BMGGWMYS78NG817.md
title: 20210404 - 如何给LWP降速
indexed_at: 2026-02-27T10:00:08.786025+00:00
---

用户要求我分析一篇关于 LWP (Light Weight Process, 轻量级进程/线程) 降速的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：尝试对 LWP/线程进行 CPU 限速
2. **尝试1**：研究 cpulimit 原理 - 通过 SIGSTOP/SIGCONT 控制进程，但发现信号发送给线程会导致整个进程停止
3. **尝试2**：查阅 POSIX 线程文档，发现 SIGSTOP 发送给线程会停止整个进程的所有线程
4. **其他方案**：提到 GDB 可以停止单个线程，关联到另一篇笔记

**标签**：LWP、线程限速、SIGSTOP、cpulimit、POSIX 线程、Linux

**摘要**：记录了尝试对 LWP(轻量级进程) 进行 CPU 限速的两种失败方案。cpulimit 和 pthread_kill 发送 SIGSTOP 都会导致整个进程停止而非单个线程。

**关键概念**：
- cpulimit: 通过 procfs 读取 cpu_time，用 SIGSTOP/SIGCONT 限速进程的工具
- SIGSTOP/SIGCONT: 进程控制信号，但对线程使用会影响整个进程
- POSIX 线程: pthread_kill 发送信号会影响整个进程而非单个线程
- LWP: 轻量级进程，即 Linux 中的线程

**关联笔记**：
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb只停止触发断点的线程 - 笔记中明确提到这个关联，是同一天关于 GDB 停止单个线程的研究
## 标签
LWP, 线程限速, SIGSTOP, cpulimit, POSIX 线程, Linux

## 摘要
记录了尝试对 LWP(轻量级进程) 进行 CPU 限速的两种失败方案。cpulimit 和 pthread_kill 发送 SIGSTOP 都会导致整个进程停止而非单个线程。最终转向研究 GDB 停止单个线程的方案。

## 关键概念
- cpulimit: 通过 procfs 读取 cpu_time，用 SIGSTOP/SIGCONT 控制进程的限速工具
- SIGSTOP/SIGCONT: 进程控制信号，作用于进程级别，发送给线程会影响整个进程
- pthread_kill: POSIX 线程信号发送函数，但信号行为仍作用于进程层面
- LWP: 轻量级进程，Linux 内核中线程的实现方式

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 笔记中明确引用，记录 GDB 如何只停止触发断点的单个线程
