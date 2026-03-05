---
note: 01KJBZNKZMMHGYY4D6PE7BBQMW.md
title: 20241209 - 使用ms-swift对Qwen2-VL进行微调 [12] - 调整loss函数
indexed_at: 2026-03-05T11:05:37.029338+00:00
---

## 摘要
记录对 Qwen2-VL 微调时尝试将 JSON 输出中的数字字段使用 MSE loss、selfies 字段使用交叉熵 loss 的混合 loss 方案。探索了保持梯度可微的技术方案（softmax+logits 直接计算），发现非法 JSON 格式对 loss 计算影响过大，需通过 few-shot 增强 JSON 格式稳定性。

## 关键概念
- 混合 Loss 函数: 对输出中不同类型字段（数字/文本）使用不同 loss 计算方式（MSE/交叉熵）
- 梯度可微性: loss 计算需直接使用 logits 和 softmax，避免 argmax/decode 等不可微操作
- Label Smoothing: 为目标数字相邻值分配小概率，帮助模型理解数值连续性
- Teacher Forcing: 训练时使用真实目标 token 作为输入，加速数值部分学习
- num_items_in_batch: 训练 batch 中所有数据 labels 的有效 token 数总和，用于 loss 归一化

## 关联笔记
- 01KJBZN5YYTD5QQ8Z1D621Z88K.md: 前一篇 [11]，尝试多任务训练增强原子数理解，为本篇 loss 调整提供动机
- 01KJBZNZQ5X5K7KZJPBRPS7VTZ.md: 后一篇 [13]，采用递增式训练策略，转向课程学习方法
- 01KJBZJYMDWQXWHPAVTRVWV50H.md: 早期 [4] 尝试通过 loss_scale_config_path 调整特定 token 的 loss 权重
