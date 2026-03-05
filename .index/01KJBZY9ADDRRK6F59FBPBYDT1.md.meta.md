---
note: 01KJBZY9ADDRRK6F59FBPBYDT1.md
title: 20250815 - 阅读论文*: Qwen-Image Technical Report
indexed_at: 2026-03-05T11:53:57.665628+00:00
---

## 标签
Qwen-Image, 图像生成, 扩散模型, MMDiT, VAE, Qwen-VL

## 摘要
Qwen-Image 技术报告详细解析了图像生成模型的架构设计：使用 Qwen2.5-VL 生成"创作蓝图"作为条件嵌入，通过 MMDiT 在潜空间进行迭代去噪，最终由 VAE Decoder 还原为高清图像。训练策略采用冻结 Qwen-VL 和 VAE、专注训练 MMDiT 的方式，并通过 SFT→DPO→GRPO 三阶段对齐优化生成质量。

## 关键概念
- MMDiT: 多模态扩散 Transformer，核心生成引擎，负责在潜空间根据文本指令对噪声图像进行迭代去噪
- 创作蓝图 (条件嵌入 h): 由冻结的 Qwen2.5-VL 将文本 Prompt 转换为的高维向量，编码详细的创作指令


- MSRoPE: 多尺度旋转位置编码，为图像块和文本词元提供统一的空间位置信息，解决 VAE 卷积导致的位置丢失问题
- 潜空间 (Latent Space): VAE 压缩后的抽象数字画布，在此空间进行扩散去噪可大幅降低计算成本
- 课程学习: 从低分辨率到高分辨率、从简单到复杂的分阶段训练策略

## 关联笔记
- 01KJBZTFWATGMK1VW5NRVDQKMP.md: 讨论扩散模型和多模态模型分类，提供 Qwen-Image 所属的技术背景
- 01KJBZTDBNG5BNXG2V0YN0W5SW.md: OpenSora 的 DiT 架构解析，与 Qwen-Image 的 MMDiT 核心引擎形成对比参考## 标签
Qwen-Image, 图像生成, 扩散模型, MMDiT, VAE, Qwen-VL

## 摘要
Qwen-Image 技术报告详细解析了图像生成模型的架构设计：使用 Qwen2.5-VL 生成"创作蓝图"作为条件嵌入，通过 MMDiT 在潜空间进行迭代去噪，最终由 VAE Decoder 还原为高清图像。训练策略采用冻结 Qwen-VL 和 VAE、专注训练 MMDiT 的方式，并通过 SFT→DPO→GRPO 三阶段对齐优化生成质量。

## 关键概念
- MMDiT: 多模态扩散 Transformer，核心生成引擎，负责在潜空间根据文本指令对噪声图像进行迭代去噪
- 创作蓝图 (条件嵌入 h): 由冻结的 Qwen2.5-VL 将文本 Prompt 转换为的高维向量，编码详细的创作指令
- MSRoPE: 多尺度旋转位置编码，为图像块和文本词元提供统一的空间位置信息，解决 VAE 卷积导致的位置丢失问题
- 潜空间 (Latent Space): VAE 压缩后的抽象数字画布，在此空间进行扩散去噪可大幅降低计算成本
- 课程学习: 从低分辨率到高分辨率、从简单到复杂的分阶段训练策略

## 关联笔记
- 01KJBZTFWATGMK1VW5NRVDQKMP.md: 讨论扩散模型和多模态模型分类，提供 Qwen-Image 所属的技术背景
- 01KJBZTDBNG5BNXG2V0YN0W5SW.md: OpenSora 的 DiT 架构解析，与 Qwen-Image 的 MMDiT 核心引擎形成对比参考
