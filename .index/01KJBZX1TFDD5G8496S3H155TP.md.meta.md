---
note: 01KJBZX1TFDD5G8496S3H155TP.md
title: 20250810 - 使用nsys, 分析A100 8卡的时间消耗
indexed_at: 2026-02-27T11:44:28.622571+00:00
---

用户要求我分析这篇关于使用 NVIDIA Nsight Systems 分析 A100 8 卡时间消耗的笔记，并生成索引元数据。

让我分析笔记内容：
1. 主题：使用 Nsight Systems (nsys) 分析 GPU 训练性能
2. 涉及技术：NVIDIA Nsight Systems, A100 GPU, 深度学习训练，ms-swift, torch.distributed
3. 主要内容：远程连接、命令行执行、性能分析报告、CUDA kernel 分析

标签应该包括：性能分析、GPU、Nsight Systems、深度学习训练等

摘要：记录使用 Nsight Systems 分析 A100 8 卡训练时间消耗的过程，包括远程连接方式和报告解读。

关键概念：Nsight Systems、CUDA kernel、分布式训练等

关联笔记：查看提供的笔记列表，主要是 MySQL、Clickhouse、性能分析相关的，但这篇笔记是关于 GPU 性能分析的，与现有的笔记（主要是数据库相关）没有直接关联。

让我生成输出。## 标签
性能分析，GPU, Nsight Systems, 深度学习训练，A100, 分布式训练

## 摘要
记录使用 NVIDIA Nsight Systems 分析 A100 8 卡训练任务时间消耗的过程。包括通过 SSH 远程连接执行 ms-swift 训练命令，以及解读 nsys 报告中的 CUDA kernel 和 Graph 运行情况。

## 关键概念
- Nsight Systems (nsys): NVIDIA 性能分析工具，用于分析 GPU 应用的时间线
- CUDA Kernel: GPU 上并行执行的计算函数，在 nsys 报告中以蓝色显示
- torch.distributed.run: PyTorch 分布式训练启动器，用于多卡训练
- GRPO: 强化学习训练算法，笔记中使用的训练类型

## 关联笔记
无
