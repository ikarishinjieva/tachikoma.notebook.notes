---
note: 01KJBYXQHSCD3DVCRPTGQVN88W.md
title: 20220910 - 百胜复制延迟, 再次整理线索 - 中断
indexed_at: 2026-03-05T08:23:33.617665+00:00
---

## 摘要
记录百胜复制延迟问题中通过 turbostat 和 dstat 工具发现 CAL/LOC 中断异常高的诊断过程，结合 Linux 源码分析 smp_call_function_interrupt 机制及 IPI 触发来源。使用 BCC 工具（trace-bpfcc、stackcount-bpfcc）追踪 CAL 中断的生产方，定位 NUMA-0 IRQ 分布不均问题。

## 关键概念
- CAL (Function Call Interrupt): 函数调用中断，由 smp_call_function_interrupt 处理，统计值通过 inc_irq_stat(irq_call_count) 累加
- IPI (Inter-Processor Interrupt): 处理器间中断，用于多核间通信，通过 CALL_FUNCTION_VECTOR 触发
- NUMA: 非统一内存访问架构，IRQ 分布不均可能导致性能问题
- turbostat: Linux 工具，解析 /proc/interrupts 监控 CPU 状态和硬件 IRQ
- smp_call_function_many: 内核函数，触发多核函数调用中断，与 TLB flush 等操作相关

## 关联笔记
- 01KJBYJV26733NF0YPZ7ETDET7.md: 同周期的性能分析方法笔记，涉及 perf 和 Vtune 指标分析
- 01KJBYEJ6EH6E3EXVZ6D3M5TWH.md: THP 与 TLB 性能优化文档，涉及类似的内核性能调优场景
- 01KJBYEK8DZVDASX1CPQPXC5DA.md: TLB 性能测试方法，与 CAL/TLB 中断统计分析相关
