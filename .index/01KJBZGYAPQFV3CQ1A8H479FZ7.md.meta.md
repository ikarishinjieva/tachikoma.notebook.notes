---
note: 01KJBZGYAPQFV3CQ1A8H479FZ7.md
title: 20241016 - 阅读论文*: Img2Mol – accurate SMILES recognition from molecular graphical depictions†
indexed_at: 2026-03-05T10:48:36.995589+00:00
---

## 摘要
论文提出将分子图像到 SMILES 的任务拆解为图像到 CDDD 向量、CDDD 向量到 SMILES 两个阶段。笔记还整理了其他分子嵌入技术如 VAE、GAN、GNN、ECFPs 等作为参考。

## 关键概念
- Img2Mol: 从分子结构图像识别生成 SMILES 字符串的深度学习模型
- CDDD 向量: 连续离散深度学习描述符，用于分子的连续向量表示
- SMILES: 分子结构的线性文本表示法
- 分子嵌入: 将分子结构转换为连续向量的技术
- 图神经网络: 直接在分子图上操作学习分子表示的深度学习模型

## 关联笔记
- 01KJBZH3679KRGP1N7E91WZR91.md: 记录训练 Img2Mol 项目的过程和遇到的问题
- 01KJBZGZS514G9FB33XS2BKZ97.md: 记录测试和安装 Img2Mol 项目的过程及代码解读
- 01KJBZJ4ZJYX6CE5DC0AKEKGHH.md: 使用 Img2Mol 中的图片变换方法
