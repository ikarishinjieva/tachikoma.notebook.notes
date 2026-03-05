---
note: 01KJBYEJ6EH6E3EXVZ6D3M5TWH.md
title: 20210828 - THP 文档
indexed_at: 2026-03-05T07:42:29.012719+00:00
---

## 摘要
记录 THP（透明大页）的配置方式、缺陷及调优建议，包括 enabled/defrag/khugepaged 等核心参数。分析 THP 对 TLB 命中率的影响，指出数据库和延迟敏感场景建议关闭 THP，默认推荐 defer+madvise 策略。

## 关键概念
- THP (Transparent Huge Pages): Linux 内核将多个 4KB 页合并为 2MB 大页，减少 TLB miss
- TLB (Translation Lookaside Buffer): CPU 缓存虚拟地址到物理地址的转换，命中率影响性能
- defrag: THP 内存碎片整理策略，控制无法分配大页时的行为
- khugepaged: 内核线程，负责后台将 4KB 页合并为 2MB 大页
- MADV_HUGEPAGE: 内存标记，用于对特定内存区域启用 THP 功能

## 关联笔记
- 01KJBYEK8DZVDASX1CPQPXC5DA.md: 同系列笔记，探索 THP 对 TLB 的作用及 perf 测试方法
- 01KJBYKEEAEBE19H8NX3SFJXR3.md: 分析 NUMA 和 THP 配合导致的性能下降及 TLB miss 指标
- 01KJBYXQHSCD3DVCRPTGQVN88W.md: 讨论 TLB 统计值来源及与 NUMA/PTE/THP 相关的内核 patch
