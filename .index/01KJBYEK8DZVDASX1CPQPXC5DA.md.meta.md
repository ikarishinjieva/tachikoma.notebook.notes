---
note: 01KJBYEK8DZVDASX1CPQPXC5DA.md
title: 20210830 - THP的作用探索
indexed_at: 2026-03-05T07:42:51.906158+00:00
---

## 摘要
通过 C 代码和 sysbench 测试 THP 对 TLB miss 的影响，使用 perf 统计 dTLB-load/store-misses 和 page walk 耗时。THP 开启后 TLB miss 导致的 page walk 成本从 21% 降至 3.4%，执行时间减少 33%。

## 关键概念
- THP (Transparent Hugepage): Linux 透明大页机制，将多个 4KB 页合并为 2MB 大页以减少 TLB miss
- TLB (Translation Lookaside Buffer): 页表缓存，加速虚拟地址到物理地址的转换
- page walk: TLB miss 时硬件遍历页表进行地址转换的过程，耗时较高
- dTLB: 数据 TLB，缓存数据访问的页表项，分为 loads 和 stores
- perf stat: Linux 性能分析工具，可统计 CPU cycles、instructions、TLB miss 等硬件事件

## 关联笔记
- 01KJBYEJ6EH6E3EXVZ6D3M5TWH.md: THP 配置文档，包含 THP 相关配置参数和缺陷说明
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 内存页表诊断笔记，涉及 TLB、PMH 和 page walk 原理
