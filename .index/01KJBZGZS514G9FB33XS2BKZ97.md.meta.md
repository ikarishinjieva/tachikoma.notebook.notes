---
note: 01KJBZGZS514G9FB33XS2BKZ97.md
title: 20241016 - 测试Img2Mol项目
indexed_at: 2026-03-05T10:48:53.874189+00:00
---

## 摘要
记录 Img2Mol 项目的安装配置过程及遇到的问题解决方案，包括 RDKit 版本冲突修复和本地 CDDD 模型部署。分析 Img2MolInference 推理流程和 Img2MolPlModel 模型结构，解释图像张量变换与卷积分类器架构。

## 关键概念
- Img2MolInference: 从分子图像预测 SMILES 表达式的推理类，通过 CNN 生成 CDDD 向量再解码
- CDDD 向量: 512 维连续描述符向量，作为分子结构的中间表示，可转换为 SMILES
- 图像张量变换: 将 PIL 图像 (H×W×C) 转换为 PyTorch 张量 (C×H×W)，支持数据增强和批量处理
- 卷积器 + 分类器: Img2Mol 模型架构，卷积层提取图像特征，全连接层输出 CDDD 向量
- RDKit: 化学信息学库，用于 SMILES 与分子结构之间的转换和验证

## 关联笔记
- 01KJBZGYAPQFV3CQ1A8H479FZ7.md: 阅读 Img2Mol 论文，详细分析数据集构建和模型设计
- 01KJBZH3679KRGP1N7E91WZR91.md: 训练 Img2Mol 项目的实践记录和评估结果
- 01KJBZGTZEAQ1EWNGJNQTJRNBR.md: 提及 Img2Mol 作为 smile to image 模型的参考
