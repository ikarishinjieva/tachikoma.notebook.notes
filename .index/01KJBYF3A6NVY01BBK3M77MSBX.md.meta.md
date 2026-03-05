---
note: 01KJBYF3A6NVY01BBK3M77MSBX.md
title: 20211013 - 探索 dynimize 原理
indexed_at: 2026-03-05T07:47:41.943254+00:00
---

## 摘要
通过 strace 分析 dynimize 工具的工作原理，发现其使用 ptrace 系统调用对 MySQL 进程进行内存注入和代码修改。同时通过修改/proc/sys/kernel/perf 相关配置和调整进程 nice 值来优化性能。

## 关键概念
- ptrace: Linux 进程跟踪系统调用，用于附加、读写目标进程内存和寄存器
- strace: 系统调用追踪工具，用于分析程序的底层系统行为
- process_vm_readv/writev: 跨进程内存读写系统调用
- 内存注入: 通过 PTRACE_POKEDATA 向目标进程代码段写入自定义代码
- nice 优先级: Linux 进程调度优先级，dyni 将自身调整为 -20 最高优先级

## 关联笔记
- 01KJBYF973H98W1RNZ05SS50KE.md: 基于此笔记分析的 dynimize 原理实现 opt4db 项目
