---
note: 01KJBYF7W9WSV2F1EH10C6RVQW.md
title: 20211108 - gdb追踪fork出的进程
indexed_at: 2026-02-27T10:13:04.368433+00:00
---

用户要求我分析一篇关于 gdb 追踪 fork 进程的笔记，并生成索引元数据。让我分析笔记内容：

笔记标题：20211108 - gdb 追踪 fork 出的进程

内容主要是关于使用 gdb 调试 fork 出的子进程的命令配置。

标签应该包括：gdb, fork, 进程调试, 子进程追踪等

摘要：介绍如何使用 gdb 追踪 fork 出的子进程，需要设置特定的调试选项。提到了 gdb 版本兼容性问题。

关键概念：
- set detach-on-fork: 控制 gdb 是否分离 fork 出的进程
- set follow-fork-mode: 控制 gdb 跟踪父进程还是子进程
- gdb 版本兼容性：gdb 8 与 gdb 10.2 的差异

关联笔记：
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb 只停止触发断点的线程 - 都是 gdb 调试相关的笔记
- 01KJBYD9RN1BMGGWMYS78NG817.md: 20210404 - 如何给 LWP 降速 - 也是调试相关的

让我生成输出。## 标签
gdb, fork, 进程调试, 子进程追踪, 版本兼容

## 摘要
介绍使用 gdb 追踪 fork 出的子进程的调试配置方法。需设置 `detach-on-fork off` 和 `follow-fork-mode child` 才能断点到子进程的 main 函数。gdb 8 存在兼容性问题，需升级到 gdb 10.2。

## 关键概念
- set detach-on-fork: 控制 gdb 是否分离 fork 出的进程，off 表示保持附加
- set follow-fork-mode: 指定 fork 后跟踪父进程还是子进程，child 表示跟踪子进程
- gdb 版本差异: 不同版本对 fork 调试支持不同，旧版本可能报错

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 同为 gdb 调试配置相关笔记，涉及线程断点行为
- 01KJBYD9RN1BMGGWMYS78NG817.md: 同为底层调试技术笔记，涉及 LWP 线程控制
