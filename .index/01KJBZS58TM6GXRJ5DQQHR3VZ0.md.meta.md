---
note: 01KJBZS58TM6GXRJ5DQQHR3VZ0.md
title: 20250319 - 研究SQL优化模型微调 - 靠结构流程影响优化流程
indexed_at: 2026-03-05T11:35:20.684981+00:00
---

## 标签
SQL 优化，模型微调，DeepSeek-R1, SFT, DDP 训练，结构流程

## 摘要
记录使用 4 GPU + DDP 对 DeepSeek-R1-Distill-Qwen-7B 进行 SFT 微调的训练配置和参数。核心目标是通过"结构流程"和"优化流程"两套数据联合训练，让模型建立两者关联，并验证模型对未见 SQL 的优化流程生成能力。

## 关键概念
- DDP (Distributed Data Parallel): PyTorch 分布式训练框架，支持多 GPU 并行训练加速
- SFT (Supervised Fine-Tuning): 监督微调，使用标注数据对预训练模型进行任务适配
- 结构流程: 对 SQL 语句的结构特征进行分析的流程数据
- 优化流程: 基于结构分析生成 SQL 优化方案的流程数据
- LoRA: 低秩适配器，参数高效的微调方法，减少训练资源消耗

## 关联笔记
- 01KJBZS11VJFPQH5KCKTTBJ8K7.md: 前继笔记，探索 SQL 优化模型训练的不同方法和技术方案
- 01KJBZS64F0K9J2QBFYQNS6ETM.md: 后续验证，证明结构流程可以增强优化流程的训练效果
