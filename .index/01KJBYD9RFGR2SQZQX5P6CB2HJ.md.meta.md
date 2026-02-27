---
note: 01KJBYD9RFGR2SQZQX5P6CB2HJ.md
title: 20210404 - gdb只停止触发断点的线程
indexed_at: 2026-02-27T09:59:56.521241+00:00
---

用户要求我分析一篇关于 GDB 调试的笔记，生成索引元数据。让我分析笔记内容：

笔记标题：20210404 - gdb 只停止触发断点的线程

笔记内容涉及 GDB 调试配置：
- set target-async 1 - 异步目标模式
- set pagination off - 关闭分页
- set non-stop on - 非停止模式（只停止触发断点的线程）
- interrupt -a - 停止整个程序

其他笔记列表中，有一篇可能相关：
- 01KJBYD9RN1BMGGWMYS78NG817.md: 20210404 - 如何给 LWP 降速 (LWP 是 Lightweight Process，与线程调试可能相关，同一天)

现在生成元数据：

标签：GDB, 调试，多线程，断点，non-stop 模式

摘要：介绍 GDB 非停止模式 (non-stop) 的配置方法，该模式下只有触发断点的线程会停止。其他线程继续运行，可使用 interrupt -a 停止整个程序。

关键概念：
- non-stop 模式：GDB 调试模式，只停止触发断点的线程
- target-async：GDB 异步目标模式
- interrupt -a：GDB 命令，停止所有线程

关联笔记：
- 01KJBYD9RN1BMGGWMYS78NG817.md: 同一天 (20210404) 的线程相关调试笔记，涉及 LWP（轻量级进程）## 标签
GDB, 调试，多线程，断点，non-stop 模式，线程控制

## 摘要
介绍 GDB 非停止模式 (non-stop) 的配置方法，该模式下只有触发断点的线程会停止，其他线程继续运行。提供了完整的配置命令及如何停止整个程序的方法。

## 关键概念
- non-stop 模式：GDB 调试模式，只停止触发断点的线程而非整个进程
- target-async：启用 GDB 异步目标模式，支持非阻塞调试
- interrupt -a：GDB 命令，强制停止所有线程

## 关联笔记
- 01KJBYD9RN1BMGGWMYS78NG817.md: 同为 20210404 的线程调试相关笔记，涉及 LWP（轻量级进程）控制
