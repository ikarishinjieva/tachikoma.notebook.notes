---
note: 01KJBYXQKER6WW6C71KQ4VHRPX.md
title: 20220818 - 百胜slave延迟, 重新整理所有线索
indexed_at: 2026-02-27T10:23:30.426609+00:00
---

用户要求我分析一篇关于百胜 slave 延迟问题的技术笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 MySQL slave 延迟诊断的笔记
2. 涉及的技术点包括：CPU 架构（Skylake）、NUMA、cstate、perf 性能分析、页表、THP（透明大页）等
3. 核心问题是：slave 延迟与空闲连接、CPU 架构、cstate 禁用等因素相关
4. 有关联的其他百胜延迟诊断笔记

让我生成符合要求的索引：

标签应该包括：MySQL、slave 延迟、性能诊断、NUMA、CPU 架构等

摘要需要概括核心内容：这篇笔记记录了百胜 MySQL slave 延迟问题的诊断过程，通过分析发现空闲连接、CPU 架构（Skylake）和 cstate 设置是主要影响因素。

关键概念：
- NUMA: 非统一内存访问架构，影响内存分布性能
- cstate: CPU 省电状态，禁用后可避免延迟
- THP: 透明大页，可能影响数据库性能
- IPC: 每周期指令数，延迟时下降
- dcycles: 延迟周期比例

关联笔记：
- 20220808 - 百胜 slave 延迟，关于内存页表的相关诊断（同一问题的页表分析）


- 20220818 - 百胜 slave 延迟，关于上下文切换的诊断（同一天的上下文切换分析）
- 20220612 - numa blancing（NUMA 相关）
## 标签
MySQL, slave 延迟，性能诊断，NUMA, CPU 架构，cstate

## 摘要
记录百胜 MySQL slave 延迟问题的诊断过程，通过 perf 工具分析发现空闲连接、Skylake CPU 架构和 Intel Idle driver 是主要影响因素。禁用 cstate 和清理空闲连接后延迟消失，同时观察到 IPC 下降、dcycles 比例升高、numa 分布不均等现象。

## 关键概念
- NUMA: 非统一内存访问架构，内存分布不均会影响性能
- cstate: CPU 省电状态，禁用 Intel Idle driver 可避免延迟
- THP: 透明大页，可能影响数据库性能需检查
- IPC: 每周期指令数，延迟发生时下降一半
- dcycles: 延迟周期比例，有延迟时约 10%，正常约 5%

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 同属百胜 slave 延迟问题系列诊断笔记（内存页表相关）
- 01KJBYD9RN1BMGGWMYS78NG817.md: 同属百胜 slave 延迟问题系列诊断笔记（上下文切换相关）
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存相关分析，与 NUMA 和内存分布问题有关联
