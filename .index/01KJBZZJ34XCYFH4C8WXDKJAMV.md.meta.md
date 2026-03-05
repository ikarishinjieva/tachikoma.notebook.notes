---
note: 01KJBZZJ34XCYFH4C8WXDKJAMV.md
title: 20250903 - 阅读论文*: RS-Lora: A Rank Stabilization Scaling Factor for Fine-Tuning with LoRA
indexed_at: 2026-03-05T11:56:05.282250+00:00
---

## 标签
LoRA, 秩稳定, rsLoRA, 缩放因子, 梯度崩溃, 微调

## 摘要
论文发现标准 LoRA 的缩放因子α/r 导致高秩时梯度崩溃、性能停滞，提出秩稳定概念。rsLoRA 将缩放因子改为α/√r，使前向/反向传播数值量级不随秩变化而波动，实现高秩稳定训练。

## 关键概念
- 秩稳定 (Rank-Stabilized): 适配器无论秩 r 如何变化，前向传播数据和反向传播梯度的数值量级都保持稳定
- rsLoRA: 秩稳定 LoRA，核心改动是将缩放因子从α/r 修正为α/√r
- 秩 (Rank): LoRA 两矩阵 BA 的中间衔接维度数，r 越大参数量越多但标准 LoRA 训练越不稳定
- 缩放因子: 决定 BA 对权重矩阵影响力的系数，影响前向传播和反向传播的梯度传递

## 关联笔记
- 01KJBZZHDRJBS1FEX3M60AZ305.md: 同一天阅读的 LoRA 改进论文 (DoRA)，分析 LoRA 与全量微调差距并提出权重分解方案
- 01KJBZSRBWRW506KCEADZ31H61.md: 使用 LoRA 进行 SQL 优化模型微调的实践记录，包含 lora_rank 等参数配置
