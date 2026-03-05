---
note: 01KJBZTFWATGMK1VW5NRVDQKMP.md
title: 20250610 - 分析混元模型
indexed_at: 2026-03-05T11:43:53.764913+00:00
---

## 标签
HunyuanVideo, 扩散模型, 多模态学习，视频生成，Transformer, 数据预处理

## 摘要
分析腾讯混元视频生成模型 (HunyuanVideo) 的技术架构，包括扩散模型与 Transformer 结合的混合架构设计。详细介绍数据预处理流程（分层过滤、结构化字幕、摄像机运动分类）和模型工作流程（因果 3D VAE、双流 Transformer、时空分块策略）。

## 关键概念
- 扩散模型：通过逐步加噪和逆转去噪过程生成数据的模型架构


- 因果 3D VAE：捕捉视频时空动态的压缩编码器
- 结构化字幕：VLM 生成的多维度 JSON 格式视频描述
- 混合 Transformer：双流处理单模态特征后合并进行跨模态交互
- 时空分块策略：将视频分解成小块以降低计算资源需求

## 关联笔记
- 01KJBZTC50Q9B29PH1TZ193P7E.md: 视频生成模型学习笔记，包含 HunyuanVideo 项目链接
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: Qwen-Image 技术报告，涉及扩散模型和 MMDiT 架构对比## 标签
HunyuanVideo, 扩散模型, 多模态学习，视频生成，Transformer, 数据预处理

## 摘要
分析腾讯混元视频生成模型 (HunyuanVideo) 的技术架构，包括扩散模型与 Transformer 结合的混合架构设计。详细介绍数据预处理流程（分层过滤、结构化字幕、摄像机运动分类）和模型工作流程（因果 3D VAE、双流 Transformer、时空分块策略）。

## 关键概念
- 扩散模型：通过逐步加噪和逆转去噪过程生成数据的模型架构
- 因果 3D VAE：捕捉视频时空动态的视频压缩编码器
- 结构化字幕：VLM 生成的多维度 JSON 格式视频描述
- 混合 Transformer：双流处理单模态特征后合并进行跨模态交互
- 时空分块策略：将视频分解成时空小块以降低计算资源需求

## 关联笔记
- 01KJBZTC50Q9B29PH1TZ193P7E.md: 视频生成模型学习笔记，包含 HunyuanVideo 项目链接
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: Qwen-Image 技术报告，涉及扩散模型和 MMDiT 架构对比
