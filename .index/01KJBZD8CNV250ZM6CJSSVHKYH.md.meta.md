---
note: 01KJBZD8CNV250ZM6CJSSVHKYH.md
title: 20240807 - 查看python的进程堆栈
indexed_at: 2026-03-05T10:29:41.684321+00:00
---

## 摘要
记录使用 py-spy 工具查看 Python 进程堆栈的方法。通过 pip 安装后，使用 `py-spy dump --pid` 命令即可获取指定进程的执行堆栈信息。

## 关键概念
- py-spy: Python 采样性能分析工具，可无需重启进程即可查看堆栈
- dump 命令: 输出目标进程当前时刻的调用堆栈快照
- pid: 进程标识符，用于指定要分析的目标进程

## 关联笔记
- 01KJC02BRZ0WYFF71HAK5SAJGA.md: 同样使用 py-spy 进行 ROS2 节点性能分析和 OOM 问题诊断
