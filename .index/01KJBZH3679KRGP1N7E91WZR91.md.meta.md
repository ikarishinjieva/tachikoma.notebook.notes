---
note: 01KJBZH3679KRGP1N7E91WZR91.md
title: 20241018 - 训练Img2Mol项目
indexed_at: 2026-03-05T10:49:14.645238+00:00
---

## 摘要
记录训练 Img2Mol 项目时遇到的 AttributeError (tensorflow.io.gfile) 报错及修复方法。解决 TensorBoard 查看训练日志的环境配置问题，并修复验证步骤中 cddd 与 cddd_hat 张量维度不匹配的问题。

## 关键概念
- TensorBoard: 深度学习训练可视化工具，用于查看训练日志和指标
- CDDD: 连续离散分子描述符，512 维向量表示分子结构
- PyTorch Lightning: 基于 PyTorch 的深度学习训练框架
- SMILES: 分子结构的字符串表示法，Img2Mol 的最终输出目标

## 关联笔记
- 01KJBZGYAPQFV3CQ1A8H479FZ7.md: 20241016 的 Img2Mol 论文阅读笔记，提供训练的理论基础
- 01KJBZGZS514G9FB33XS2BKZ97.md: 20241016 的 Img2Mol 项目环境搭建和代码解读笔记
- 01KJBZJ4ZJYX6CE5DC0AKEKGHH.md: 20241029 的后续模型训练笔记，在新 GPU 服务器上进行分布式训练
