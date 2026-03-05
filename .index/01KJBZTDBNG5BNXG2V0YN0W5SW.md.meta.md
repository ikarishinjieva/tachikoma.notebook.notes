---
note: 01KJBZTDBNG5BNXG2V0YN0W5SW.md
title: 20250609 - OpenSora解析
indexed_at: 2026-03-05T11:43:21.769267+00:00
---

## 标签
OpenSora, DiT, 视频生成, Transformer, 3D RoPE, DC-AE

## 摘要
OpenSora 采用 Video DC-AE 自动编码器进行视频压缩，结合双流和单流混合 Transformer 架构实现文本到视频生成。3D RoPE 位置编码为特征向量添加时空位置信息，使模型能够捕捉视频中的时空关系。

## 关键概念
- Video DC-AE: 基于深度压缩自动编码器的改进版本，引入时间压缩机制降低训练和推理成本
- DiT 架构: 混合 Transformer 架构，双流块分别处理文本和视频，单流块整合特征促进跨模态交互
- 3D RoPE: 旋转位置嵌入，为特征向量添加空间和时间位置信息
- 双流处理: 视频和文本特征在各自 Transformer 流中独立并行处理
- 单流处理: 视频和文本特征合并后进行跨模态交互学习

## 关联笔记
- 01KJBZTFWATGMK1VW5NRVDQKMP.md: 混元模型同样使用 DiT 架构和 3D RoPE 进行视频生成，可对比技术差异
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: Qwen-Image 使用 MMDiT（DiT 变体）进行图像生成，架构原理相似
