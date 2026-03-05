---
note: 01KJBZS9G27XHK76W77HVN48XA.md
title: 20250326 - 阅读论文*: The Ultra-Scale Playbook: Training LLMs on GPU Clusters Ultra-Scale Playbook：在 GPU 集群上训练LLMs
indexed_at: 2026-03-05T11:36:32.435465+00:00
---

## 标签
LLM 训练，GPU 集群，内存优化，ZeRO，数据并行，张量并行

## 摘要
论文《Ultra-Scale Playbook》笔记，系统分析大模型训练中的内存分布（参数、梯度、优化器状态、激活值）及优化方法。涵盖单 GPU 内存优化（激活重计算、梯度累计）、数据并行优化（梯度同步重叠、分桶、ZeRO-1/2/3 分片策略）、张量并行和流水线并行等超大规模训练技术。

## 关键概念
- 激活值 (Activations): 前向传播中各层的输出中间结果，反向传播用于计算梯度，长序列下占用大量显存
- 梯度累计 (Gradient Accumulation): 将 batch 分成多个微批处理，累积梯度后再更新参数，减少显存峰值
- ZeRO (Zero Redundancy Optimizer): 通过优化器状态、梯度、参数的分片消除数据并行中的内存冗余
- 张量并行 (Tensor Parallelism): 对权重、梯度、激活进行分片，无需收集即可计算，解决单 GPU 无法容纳大模型问题
- 流水线并行 (Pipeline Parallelism): 将模型层分配到不同 GPU，类似 ZeRO-3 按需传输，减少单卡显存压力

## 关联笔记
- 01KJBZW8ND6Q8NTEDX7B4WWACP.md: 记录了 GRPO 训练中使用 TP/DP 配置和 ZeRO 级别的实验
- 01KJBZX5SBH11JY81B6NSAW0Y6.md: 诊断 GRPO 训练过程的时间/内存消耗，涉及显存波动分析
