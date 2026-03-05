---
note: 01KJBZX25RKXAHW2N026JFEN5X.md
title: 20250809 - A100 8卡物理机 安装
indexed_at: 2026-03-05T11:52:17.956135+00:00
---

## 摘要
记录 8 卡 A100 物理机的安装配置过程，包括 NVLink 拓扑、磁盘配置、conda 环境、ms-swift 和 vllm 依赖安装。对比了 4090 与 A100 在 GRPO 训练中的性能差异，测试了 rollout 和 colocate 两种模式的训练时长和资源配置。

## 关键概念
- NVLink Topology: GPU 间通过 NV12 连接，8 卡分为两个 NUMA 节点
- Rollout 模式: 部分 GPU 作为推理服务器，其余 GPU 用于训练
- Colocate 模式: 所有 GPU 同时参与推理和训练，通过 offload 和 sleep 节省资源
- vllm_gpu_memory_utilization: 控制 vllm 显存使用比例的关键参数

## 关联笔记
- 01KJBZW8ND6Q8NTEDX7B4WWACP.md: 同一 GRPO 训练项目的前继笔记，定义了尝试 11 的基准配置
- 01KJBZX5SBH11JY81B6NSAW0Y6.md: A100 机器用于诊断 GRPO 训练过程的时间/内存消耗分析
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 引入 A100 机器进行对比测试的后续训练记录
