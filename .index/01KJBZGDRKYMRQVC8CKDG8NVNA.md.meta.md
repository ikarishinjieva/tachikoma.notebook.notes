---
note: 01KJBZGDRKYMRQVC8CKDG8NVNA.md
title: 20241012 - 训练 "对ChatDBA的问答质量进行评论的系统"
indexed_at: 2026-03-05T10:48:02.937539+00:00
---

## 标签
ChatDBA, SFT, DPO, LLaMA-Factory, 模型训练, 问答评论

## 摘要
记录使用 LLaMA-Factory 对 Qwen2-7B 进行 SFT 和 DPO 两阶段训练，构建 ChatDBA 问答质量评论系统。包含数据准备流程、训练命令配置及裸模型与 SFT 后的效果对比。

## 关键概念
- SFT: 监督微调，使用 100 个问题的增强数据进行基础训练
- DPO: 直接偏好优化，在 SFT 基础上进行偏好对齐训练
- LLaMA-Factory: 大模型微调工具，提供统一的训练命令行接口
- LoRA: 低秩适应微调技术，rank=8, alpha=16

## 关联笔记
- 01KJBZZYFRDE3MNYM2SP16GZ3T.md: ChatDBA 代码仓库记录，同一项目的代码基础
- 01KJBZP5QNWMWNBV0WVYNQH5QF.md: ChatDBA 项目背景和技术积累说明
- 01KJBZJ4ZJYX6CE5DC0AKEKGHH.md: 同为 LLaMA-Factory 训练配置记录，可参考训练参数
