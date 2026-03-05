---
note: 01KJBYD9RFGR2SQZQX5P6CB2HJ.md
title: 20210404 - gdb只停止触发断点的线程
indexed_at: 2026-03-05T07:16:10.219698+00:00
---

## 摘要
记录 gdb 调试多线程程序时的行为特性。默认情况下 gdb 断点只停止触发断点的线程而非整个进程。通过设置 `non-stop on` 和 `target-async` 启用异步调试模式，使用 `interrupt -a` 命令可停止整个程序的所有线程。

## 关键概念
- non-stop 模式：gdb 设置，允许其他线程在断点触发时继续运行
- target-async：gdb 异步目标模式，支持非阻塞调试操作
- interrupt -a：gdb 命令，强制停止整个程序的所有线程
- 线程级断点：gdb 断点默认只影响触发该断点的特定线程
- pagination off：关闭 gdb 分页输出，便于脚本化调试

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同样使用 interrupt -a 命令进行 gdb 多线程调试
- 01KJBYD9RN1BMGGWMYS78NG817.md: 引用本笔记，讨论 gdb 停止单个线程的技术方案
