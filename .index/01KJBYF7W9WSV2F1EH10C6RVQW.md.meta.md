---
note: 01KJBYF7W9WSV2F1EH10C6RVQW.md
title: 20211108 - gdb追踪fork出的进程
indexed_at: 2026-03-05T07:48:44.223883+00:00
---

## 标签
gdb, fork, 进程调试, 断点设置, 配置选项

## 摘要
记录使用 gdb 追踪 fork 系统调用创建子进程的调试方法。通过设置 `detach-on-fork off`和`follow-fork-mode child` 两个关键配置，使 gdb 能够追踪子进程而非默认脱离。

## 关键概念
- detach-on-fork: 控制 gdb 在 fork 后是否脱离子进程，设为 off 可同时调试父子进程
- follow-fork-mode: 指定 fork 后 gdb 跟随哪个进程，可选 parent 或 child
- 断点设置: 在 main 函数处设置断点以捕获子进程的入口

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 使用 gdb 分析 MySQL coredump，涉及 gdb 的 bt、info reg 等调试命令
- 01KJBZHBBJKJDPNE503NXFX50V.md: Python 调试方法，使用 pdb 设置断点进行调试
