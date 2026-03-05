---
note: 01KJBZ2206KPRE6YESGGMNHKNC.md
title: 20230721 - p-tuning进行微调
indexed_at: 2026-03-05T08:53:56.358475+00:00
---

## 标签
P-Tuning, ChatGLM-6B, 微调, SQL 规则校验, LoRA 对比, 训练日志

## 摘要
记录使用 P-Tuning 对 ChatGLM-6B 进行 SQL 规则校验微调的实验过程。使用与 LoRA 相同参数效果较差，loss 波动大（1.5-2.7），模型输出质量明显不如 LoRA。

## 关键概念
- P-Tuning: 参数高效的微调方法，通过可学习的连续前缀向量引导模型
- ChatGLM-6B: 清华开源的双语对话式语言模型，本实验的基座模型
- LoRA: 低秩适配器微调方法，本实验中作为对比方案
- 损失函数 (Loss): 训练过程中衡量模型预测误差的指标，本实验中波动较大

## 关联笔记
- 01KJBZ1Z4PSVWAT3RBRKWB688P.md: p-tuning 微调的前序实验记录（20230717）
- 01KJBZ23YC0VMQ76EFC97D2ZJX.md: 从 p-tuning 换回 lora 的后续实验（20230723）
