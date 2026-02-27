---
note: 01KJBYEJ6EH6E3EXVZ6D3M5TWH.md
title: 20210828 - THP 文档
indexed_at: 2026-02-27T10:09:58.679546+00:00
---

用户要求我分析这篇关于 THP (Transparent Huge Pages) 的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：THP (Transparent Huge Pages) 透明大页
2. 主要内容：
   - 如何检查进程级别 THP 使用量
   - THP 的目的（改善 TLB 命中率）
   - THP 配置选项（enabled, defrag, khugepaged）
   - THP 的缺陷
   - THP 配置建议
   - TLB 相关解释
   - NUMA 环境下 HugePage 可能导致性能下降

3. 相关概念：THP, TLB, Huge Pages, 内存碎片整理，NUMA, 数据库性能

4. 关联笔记：从列表中看，"20210429 - 内存去哪儿了" (01KJBYDAF12RN18BECZG1QSTEC.md) 可能与内存相关，还有 "20210628 - VSS/RSS/USS/PSS 解释" (01KJBYDNQ4TQXJPQCVEH94QW3E.md) 也是内存相关的。

现在生成索引：

标签：THP, 内存管理, Linux, 性能优化, 数据库, Huge Pages

摘要：这篇笔记整理了 Linux Transparent Huge Pages (THP) 的配置、原理和对数据库性能的影响。包含 THP 检查方法、配置选项、缺陷分析以及针对数据库场景的配置建议。

关键概念：
- THP (Transparent Huge Pages): Linux 透明大页机制，将 4KB 页合并为 2MB 大页以改善 TLB 命中率
- TLB (Translation Lookaside Buffer): 页表缓存，用于加速虚拟地址到物理地址的转换
- khugepaged: 内核线程，负责后台内存碎片整理和页合并
- defrag: 内存碎片整理配置，决定无法分配 THP 时是否进行碎片整理

关联笔记：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同属内存管理主题）
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（同属内存管理主题）## 标签
THP, 内存管理, Linux, 性能优化, 数据库, Huge Pages

## 摘要
这篇笔记整理了 Linux Transparent Huge Pages (THP) 的配置、原理和对数据库性能的影响。包含 THP 检查方法、配置选项、缺陷分析以及针对数据库场景的配置建议。

## 关键概念
- THP (Transparent Huge Pages): Linux 透明大页机制，将 4KB 页合并为 2MB 大页以改善 TLB 命中率
- TLB (Translation Lookaside Buffer): 页表缓存，用于加速虚拟地址到物理地址的转换
- khugepaged: 内核线程，负责后台内存碎片整理和页合并
- defrag: 内存碎片整理配置，决定无法分配 THP 时是否进行碎片整理
- NUMA: 非统一内存访问架构，HugePage 在 NUMA 环境下可能导致跨区访问性能下降

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同属内存管理主题，探讨内存使用情况
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 同属内存管理主题，解释各类内存指标
