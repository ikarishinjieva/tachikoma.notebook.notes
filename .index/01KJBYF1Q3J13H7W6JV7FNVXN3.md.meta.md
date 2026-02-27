---
note: 01KJBYF1Q3J13H7W6JV7FNVXN3.md
title: 20211008 - 如何调试jemalloc对unwind库的调用
indexed_at: 2026-02-27T10:12:09.779106+00:00
---

用户要求我分析这篇关于调试 jemalloc 对 unwind 库调用的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：使用 gdb 调试 jemalloc 和 tcmalloc 对 unwind 库的调用
2. **核心内容**：
   - 用 jemalloc 启动 MySQL 的方法
   - 调试 jemalloc 调用 unwind 的方法（gdb 设置）
   - 调试 tcmalloc 调用 unwind 的方法
   - 展示了调用堆栈示例

3. **标签**：jemalloc, tcmalloc, gdb 调试, MySQL, 内存分析, unwind

4. **关键概念**：
   - jemalloc: 一个通用的内存分配器
   - tcmalloc: Thread-Caching Malloc，Google 开发的内存分配器
   - _Unwind_Backtrace: 用于获取调用堆栈的函数
   - LD_PRELOAD: 环境变量，用于在程序加载前预加载共享库

5. **关联笔记**：
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 (内存分析相关)
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 (MySQL 性能分析相关)
   - 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb 只停止触发断点的线程 (gdb 调试相关)
   
现在需要按照指定格式整理输出这些元数据。## 标签
jemalloc, tcmalloc, gdb 调试, MySQL, 内存分配器, 调用堆栈

## 摘要
记录使用 gdb 调试 jemalloc 和 tcmalloc 对 unwind 库调用的方法。通过设置 LD_PRELOAD 预加载内存分配器，并在 gdb 中对 _Unwind_Backtrace 设断点来监控内存分配时的调用堆栈。

## 关键概念
- LD_PRELOAD: 环境变量，用于在程序执行前预加载共享库
- _Unwind_Backtrace: libgcc 提供的函数，用于获取调用堆栈
- MALLOC_CONF: jemalloc 的配置环境变量，可设置 profiling 等参数
- TCMALLOC_STACKTRACE_METHOD: tcmalloc 指定堆栈追踪方法的配置项

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同为内存分析主题）
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb 只停止触发断点的线程（同为 gdb 调试技巧）
