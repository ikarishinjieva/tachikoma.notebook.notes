---
note: 01KJBYF0S1KGHWDAD4XA57BJNG.md
title: 20210929 - 测试 jemalloc
indexed_at: 2026-02-27T10:11:46.525032+00:00
---

用户希望我分析这篇关于 jemalloc 测试的笔记，并生成索引元数据。让我仔细阅读笔记内容：

主要内容：
1. jemalloc 需要编译时启用 --enable-prof 参数来启动 perf 功能
2. 提供了 jemalloc + MySQL 的启动命令示例
3. 解释了 lg_prof_interval 参数（每 2^26 字节分配进行一次 dump）
4. 提供了生成 svg 的命令
5. 指出 jemalloc 的 svg 导出问题：百分比错误，不记录 mmap 分配的内存量
6. 通过 perf 对比 tcmalloc 和 jemalloc 的 mmap 行为差异
7. tcmalloc 会转发 mmap，而 jemalloc 不经过，所以无法统计 mmap 分配的内存
8. 性能方面：jemalloc 开启统计时与 glibc 几乎一致

核心主题：jemalloc 内存分配器的性能分析、profiling 功能、与 tcmalloc 的对比、MySQL 场景下的使用

标签应该是：jemalloc, MySQL, 内存分析, 性能测试, tcmalloc, profiling

摘要：记录 jemalloc 在 MySQL 中的配置和使用方法，分析其 profiling 功能的限制（无法统计 mmap 分配的内存）。通过 perf 对比 tcmalloc 和 jemalloc 的内存分配行为差异。


关键概念包括 lg_prof_interval 控制 dump 间隔、LD_PRELOAD 用于预加载 jemalloc 库、perf 作为性能分析工具，以及 mmap 系统调用在两种分配器中的不同处理方式。

关联笔记中，01KJBYDAF12RN18BECZG1QSTEC.md 关于"内存去哪儿了"的主题与这篇笔记的内存分析内容最为相关，其他笔记涉及 MySQL 刷盘、gdb 调试等，关联度较低。
## 标签
jemalloc, MySQL, 内存分析, 性能测试, tcmalloc, profiling

## 摘要
记录 jemalloc 在 MySQL 中的配置方法和 profiling 功能使用，分析其无法统计 mmap 分配内存的限制。通过 perf 对比 tcmalloc 和 jemalloc 的内存分配行为差异，验证 jemalloc 不拦截 mmap 调用。

## 关键概念
- lg_prof_interval: jemalloc 参数，控制每 2^n 字节分配进行一次 heap dump
- LD_PRELOAD: Linux 环境变量，用于优先加载指定共享库
- mmap: 系统调用，直接分配内存，jemalloc 不拦截此调用
- jeprof: jemalloc 的性能分析工具，用于生成 heap 分析图表

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同为内存分析主题）
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（内存相关概念）
