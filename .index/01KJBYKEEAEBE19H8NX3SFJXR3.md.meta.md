---
note: 01KJBYKEEAEBE19H8NX3SFJXR3.md
title: 20220808 - 百胜slave延迟, 关于内存页表的相关诊断
indexed_at: 2026-02-27T10:17:51.907490+00:00
---

用户要求我分析一篇关于百胜 slave 延迟和内存页表诊断的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. **主题**：百胜 slave 延迟问题诊断，涉及内存页表、TLB、cstate 等
2. **核心发现**：
   - slave 延迟与空闲连接相关
   - cstate 未调整的 slave 页表 dcycle 消耗更高（10% vs 5%）
   - IPC 偏低与延迟相关
   - 关闭空闲连接后延迟消失

3. **技术概念**：
   - cstate/p-state（CPU 电源管理）
   - TLB（Translation Lookaside Buffer）
   - Page Table Walk
   - NUMA
   - THP（Transparent Huge Pages）
   - perf events
   - IPC（Instructions Per Cycle）

4. **关联笔记**：
   - 01KJBYDB30D5DK9MDJ70R5YFST.md: 20210513 - 百胜 cpu 高的探索（同一项目"百胜"）
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（内存相关）
   - 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（内存相关）

现在按照格式输出：
## 标签
MySQL Slave 延迟, 页表, TLB, cstate, perf 诊断, NUMA

## 摘要
记录百胜从库延迟问题的诊断过程，发现未调整 cstate 的 slave 页表 dcycle 消耗偏高（10%），且空闲连接关闭后延迟消失。分析了 TLB miss、page walk、NUMA 与 THP 配合等底层机制对性能的影响。

## 关键概念
- cstate: CPU 电源管理状态，影响处理器空闲时的功耗和唤醒延迟
- TLB: 转换旁路缓冲区，缓存虚拟地址到物理地址的翻译结果
- Page Walk: TLB 未命中时页表遍历过程，消耗 CPU 周期
- IPC: 每周期指令数，反映 CPU 执行效率
- NUMA: 非统一内存访问架构，跨节点访问会影响性能

## 关联笔记
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 同为百胜项目 CPU 性能问题探索
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存相关诊断笔记
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 内存指标 VSS/RSS/USS/PSS 解释
