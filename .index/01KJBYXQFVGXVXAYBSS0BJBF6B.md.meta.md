---
note: 01KJBYXQFVGXVXAYBSS0BJBF6B.md
title: 20220726 - TensorDB gpu 诊断
indexed_at: 2026-02-27T10:22:57.611997+00:00
---

用户要求我分析这篇关于 TensorDB GPU 诊断的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 标题：20220726 - TensorDB gpu 诊断
2. 主要内容：
   - 使用 NVIDIA Nsight Compute 诊断 TensorDB 的 GPU 使用情况
   - 记录了 TensorDB 的预训练和计算命令
   - 对比了 GPU 和非 GPU 的性能差异（GPU: ~3.4s vs 非 GPU: ~77s）
   - 详细记录了 Nsight 的使用方法和配置
   - 解释了 Occupancy Calculator 的输出

现在需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

关于关联笔记，我需要查看提供的笔记列表中是否有相关的。从列表来看，大部分是关于 MySQL、Clickhouse、内存、CPU 等主题的，与 GPU/TensorDB 诊断没有直接关联。所以关联笔记应该是「无」。

让我生成输出：## 标签
GPU 诊断，TensorDB，NVIDIA Nsight，Occupancy，性能分析，CUDA

## 摘要
记录使用 NVIDIA Nsight Compute 诊断 TensorDB GPU 性能的过程，对比 GPU 与非 GPU 模式的计算时间差异（3.4s vs 77s）。详细说明了 Nsight 的配置命令、结果查看方法，以及 Occupancy Calculator 各项指标的含义。

## 关键概念
- Occupancy: GPU SM 上活跃线程束与最大支持线程束的比率，影响并行度
- SM (Streaming Multiprocessor): GPU 的基本计算单元，包含 CUDA Cores 和线程块
- Registers Per Thread: 每个线程使用的寄存器数量，影响 occupancy
- Shared Memory: 每个 SM 可分配的共享内存，限制每 block 的线程数

## 关联笔记
无
