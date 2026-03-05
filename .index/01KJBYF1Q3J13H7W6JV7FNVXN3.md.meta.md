---
note: 01KJBYF1Q3J13H7W6JV7FNVXN3.md
title: 20211008 - 如何调试jemalloc对unwind库的调用
indexed_at: 2026-03-05T07:46:39.381809+00:00
---

## 摘要
记录使用 gdb 调试 jemalloc 和 tcmalloc 对 `_Unwind_Backtrace` 调用的方法，通过设置 `exec-wrapper` 和断点监控内存分配器的堆栈回溯行为。对比了 ltrace/latrace 等工具的局限性，提供了完整的 gdb 调试命令和调用堆栈示例。

## 关键概念
- `_Unwind_Backtrace`: libgcc 提供的堆栈回溯函数，jemalloc/tcmalloc 用它获取内存分配调用栈
- `exec-wrapper`: gdb 命令，在启动被调试程序前设置环境变量（如 LD_PRELOAD）
- `MALLOC_CONF`: jemalloc 配置环境变量，用于启用 profiling 等功能
- `TCMALLOC_STACKTRACE_METHOD`: tcmalloc 配置项，指定堆栈追踪方法（如 libgcc）

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同为 MySQL gdb 调试相关，使用 bt/where 分析崩溃堆栈
- 01KJBZ4D06APPE59ND24QFT0WQ.md: 同为 gdb 调试技术笔记，涉及断点和堆栈分析
