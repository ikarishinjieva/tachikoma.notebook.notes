---
note: 01KJBYKEEAEBE19H8NX3SFJXR3.md
title: 20220808 - 百胜slave延迟, 关于内存页表的相关诊断
indexed_at: 2026-03-05T08:05:46.150226+00:00
---

## 摘要
分析百胜 MySQL slave 延迟问题，发现未调整 cstate 的 slave 页表 dcycle 消耗高达 10%，关闭空闲连接后延迟消失。笔记整理了 perf events 含义、NUMA 与 THP 配合的性能影响及诊断命令。

## 关键概念
- C-state: CPU 空闲状态，不同级别对应不同的功耗和唤醒延迟
- TLB (Translation Lookaside Buffer): 缓存虚拟地址到物理地址的转换，miss 会触发页表遍历
- Page Table Walk: TLB miss 时硬件遍历页表获取地址转换的过程
- IPC (Instructions Per Cycle): 每周期执行的指令数，反映 CPU 执行效率
- NUMA Balancing: 自动迁移内存到访问它的 CPU 所在节点，可能引发额外 page fault

## 关联笔记
- 01KJBYXQKER6WW6C71KQ4VHRPX.md: 同一问题的后续诊断，重新整理了所有线索包括 cstate 禁用效果
- 01KJBYJTJXB03VRDH5EHQ3VQWY.md: NUMA balancing 工作机制详解，解释 page fault 与 iowait 的关系
- 01KJBYEK8DZVDASX1CPQPXC5DA.md: THP 对 TLB 影响的探索，包含 DTLB miss 的 perf 测试方法
