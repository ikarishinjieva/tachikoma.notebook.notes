---
note: 01KJBYXQHSCD3DVCRPTGQVN88W.md
title: 20220910 - 百胜复制延迟, 再次整理线索 - 中断
indexed_at: 2026-02-27T10:23:07.878544+00:00
---

用户要求我分析一篇关于 Linux 性能诊断的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：关于百胜复制延迟问题的诊断，涉及 Linux 中断机制分析
2. 核心内容：
   - 使用 turbostat 分析延迟问题
   - 发现 CAL (Function call interrupt) 和 LOC 中断计数高
   - NUMA-0 的 IRQ 高
   - 通过 BPF 工具追踪 CAL 中断来源
   - 分析 Linux 内核源码中 smp_call_function_interrupt 的机制

3. 关键概念：
   - turbostat - CPU 状态监控工具
   - CAL (Function call interrupt) - 函数调用中断
   - IRQ - 硬件中断
   - smp_call_function_interrupt - SMP 架构下的函数调用中断处理
   - NUMA - 非统一内存访问架构

4. 关联笔记：
   - 20210513 - 百胜 cpu 高的探索 (01KJBYDB30D5DK9MDJ70R5YFST.md) - 同是百胜相关的性能问题
   - 其他笔记大多是 MySQL、Clickhouse 相关，与本文 Linux 内核中断分析关联度较低

让我生成简洁的索引元数据。
## 标签
Linux 性能诊断, 中断分析, 复制延迟, turbostat, 内核调试, NUMA

## 摘要
记录百胜复制延迟问题的诊断过程，通过 turbostat 发现 CAL 和 LOC 中断计数异常。深入分析 Linux 内核源码追踪 smp_call_function_interrupt 的触发机制，使用 BPF 工具定位中断来源。

## 关键概念
- CAL (Function call interrupt): SMP 架构下 CPU 间函数调用触发的中断
- turbostat: 解析 /proc/interrupts 的 CPU 状态监控工具
- smp_call_function_interrupt: APIC 中断处理函数，用于多核间函数调用
- NUMA: 非统一内存访问架构，影响中断分布
- IPI (Inter-Processor Interrupt): 处理器间中断机制

## 关联笔记
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 同是百胜 CPU 性能问题的探索记录
