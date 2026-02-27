---
note: 01KJBZD8CNV250ZM6CJSSVHKYH.md
title: 20240807 - 查看python的进程堆栈
indexed_at: 2026-02-27T11:10:42.296608+00:00
---

用户要求我分析一篇关于 Python 进程堆栈查看的笔记，并生成索引元数据。让我分析笔记内容：

1. 笔记标题：20240807 - 查看 python 的进程堆栈
2. 笔记内容：使用 py-spy 工具来查看 Python 进程堆栈
   - pip install py-spy
   - py-spy dump --pid {pid}

其他笔记列表主要是关于 MySQL、Clickhouse、系统性能分析等主题。

我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

关于关联笔记，我需要找与 Python 调试、进程分析、性能分析相关的笔记。查看列表：
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb 只停止触发断点的线程 - 与调试相关
- 01KJBYD9RN1BMGGWMYS78NG817.md: 20210404 - 如何给 LWP 降速 - 与线程/进程相关
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - 与性能分析相关
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 与性能分析相关

这些有一些关联，都是关于性能分析/调试的。

现在生成输出。## 标签
Python, 性能分析，进程调试，py-spy, 堆栈追踪

## 摘要
介绍使用 py-spy 工具查看 Python 进程堆栈的方法。通过 pip 安装后，使用 dump 命令配合进程 ID 即可获取堆栈信息。

## 关键概念
- py-spy: Python 性能分析工具，可采样查看进程堆栈
- dump 命令: 输出指定进程某一时刻的调用堆栈
- PID: 进程标识符，用于定位目标 Python 进程

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 同为进程/线程调试工具的使用笔记（gdb）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同为性能分析相关的观测工具使用
