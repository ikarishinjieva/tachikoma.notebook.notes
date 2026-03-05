---
note: 01KJBZHWF83XQC7AE6RQD6E3YZ.md
title: 20241024 - 阅读decimer.ai的论文
indexed_at: 2026-03-05T10:52:29.149750+00:00
---

## 摘要
记录 DECIMER 项目三个核心组件：Image-Segmentation（化学式分离）、Image-Classifier（化学式检测）、Image_Transformer（图像转 SMILES）。详细分析 Encoder 采用 EfficientNet-V2 提取图像特征，Transformer Decoder 生成分子结构表示的架构设计。

## 关键概念
- DECIMER: 深度学习工具集，用于从科学文献中自动识别和提取化学结构
- EfficientNet-V2: 作为图像编码器骨干网络，将分子图像压缩为嵌入表示
- Transformer Decoder: 接收图像编码，自回归生成 SMILES 字符串序列
- Scale Factor: 控制从 EfficientNet 特征层中选择哪一层输出进行嵌入

## 关联笔记
- 01KJBZHW88F23F1VDDVAGSBKBT.md: 基于 DECIMER 架构微调 Qwen2 模型的实践计划
- 01KJBZGZS514G9FB33XS2BKZ97.md: Img2Mol 项目测试，同类分子图像识别方案对比
- 01KJBZHMH3AXJV023FRKF69ECM.md: OSRA 光学化学结构识别，传统图像处理方案参考
