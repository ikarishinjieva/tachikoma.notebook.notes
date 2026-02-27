---
note: 01KJBYF1AM3RKA6R3JM57VXRYC.md
title: 20211001 - 为什么tcmalloc会慢
indexed_at: 2026-02-27T10:12:00.436426+00:00
---

用户要求我分析一篇关于 tcmalloc 性能问题的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：探究 tcmalloc 为什么会慢
2. 核心发现：
   - tcmalloc 的 HeapProfileTable::GetCallerStackTrace 调用消耗过大
   - 与 jemalloc 对比，发现 jemalloc 有采样间隔 (lg_prof_sample 默认 512 KiB)
   - 两者都使用 libgcc 作为 backtracer 时，_Unwind_Backtrace 调用次数差异很大

3. 关键概念：
   - tcmalloc/jemalloc 内存分配器
   - HeapProfileTable::GetCallerStackTrace 堆栈跟踪
   - _Unwind_Backtrace 回溯调用
   - lg_prof_sample 采样间隔
   - libgcc/libunwind 回溯库

4. 关联笔记：
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（内存相关）
   - 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（内存相关）
   
   但这两篇主要是内存概念，与 tcmalloc 性能问题关联不强。笔记中没有直接关于 jemalloc 或 tcmalloc 的其他笔记。

让我生成简洁的索引元数据。
## 标签
tcmalloc, jemalloc, 性能分析, 内存分配器, 堆栈回溯, libgcc

## 摘要
通过修改 tcmalloc 源码定位性能瓶颈，发现 HeapProfileTable::GetCallerStackTrace 调用消耗过大。对比 jemalloc 发现其带有采样间隔机制（默认 512 KiB），而 tcmalloc 对 _Unwind_Backtrace 调用次数更多导致性能下降。

## 关键概念
- HeapProfileTable::GetCallerStackTrace: tcmalloc 中获取调用者堆栈跟踪的函数，性能消耗大
- _Unwind_Backtrace: libgcc 提供的堆栈回溯函数，频繁调用影响性能
- lg_prof_sample: jemalloc 的采样间隔参数，控制每多少字节分配进行一次采样
- libunwind: 用于堆栈回溯的库，安装后性能未见改善

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同为内存分配与性能分析主题）
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（内存相关概念）
