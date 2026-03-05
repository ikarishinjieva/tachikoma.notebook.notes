---
note: 01KJBZX1TFDD5G8496S3H155TP.md
title: 20250810 - 使用nsys, 分析A100 8卡的时间消耗
indexed_at: 2026-03-05T11:51:52.836226+00:00
---

## 摘要
记录在 Mac 上通过 SSH 连接远程 A100 8 卡机器，使用 Nsight Systems 分析 ms-swift GRPO 训练的时间消耗。展示了 nsys 命令行启动方式及生成的 CUDA kernel 和 Graph 运行时间报告，指出命令行参数缺失会导致 CUDA HW 行信息丢失。

## 关键概念
- Nsight Systems: NVIDIA 性能分析工具，用于分析 GPU 应用的时间线和资源使用
- CUDA Kernel: GPU 上执行的并行计算函数，在 nsys 时间线中以蓝色标识
- CUDA Graph: GPU 任务图，在 nsys 时间线中以橙色标识
- ms-swift: 用于大模型训练推理的框架，支持 GRPO 强化学习算法

## 关联笔记
- 01KJBZX25RKXAHW2N026JFEN5X.md: A100 8 卡物理机的安装配置，包含 NVlink 拓扑和 8 卡训练参数
- 01KJBZY39GJE38CB7BYSTRWMBX.md: A100 机器上 GRPO 训练的时间/内存占用分析，包含性能优化结论
