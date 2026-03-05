---
note: 01KJBYXQFVGXVXAYBSS0BJBF6B.md
title: 20220726 - TensorDB gpu 诊断
indexed_at: 2026-03-05T08:22:42.933520+00:00
---

## 摘要
记录使用 NVIDIA Nsight Compute 诊断 TensorDB GPU 查询性能的过程，包括 profile 命令配置和结果分析方法。详细解释 Occupancy Calculator 输出指标，分析 GPU 硬件限制因素（SM、寄存器、共享内存等）对 kernel 执行效率的影响。

## 关键概念
- Occupancy: SM 上活跃线程束数量与硬件最大值的比率，反映 GPU 资源利用率
- SM (Streaming Multiprocessor): GPU 核心计算单元，包含 CUDA Core、寄存器等资源
- Registers Per Thread: 每个线程占用的寄存器数量，影响并发线程数
- Shared Memory: 块内线程共享的高速内存，容量限制影响 occupancy
- Kernel Profiling: 使用 Nsight Compute 采集 GPU kernel 执行指标进行性能分析

## 关联笔记
- 01KJBZX1TFDD5G8496S3H155TP.md: 使用 NVIDIA Nsight Systems 分析 A100 8 卡训练时间消耗
- 01KJBZX5SBH11JY81B6NSAW0Y6.md: 使用 PyTorch profiler 诊断 GRPO 训练过程的时间/内存消耗
