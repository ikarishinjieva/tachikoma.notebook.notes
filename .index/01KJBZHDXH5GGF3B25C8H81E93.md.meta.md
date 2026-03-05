---
note: 01KJBZHDXH5GGF3B25C8H81E93.md
title: 20241021 - 研究化学论文中的截图分类
indexed_at: 2026-03-05T10:50:12.844522+00:00
---

## 标签
化学结构识别, SMILES, Img2Mol, CDDD, 多模态学习, 论文分析

## 摘要
记录使用 Gemini 和 NCI Cactus API 从化学论文截图生成 SMILES 表达式的四个样例测试，对比 Img2Mol 和 CDDD 模型的转换效果。发现 Img2Mol 直接生成SMILES错误较多，CDDD 自回归方法结果略好但 Img→CDDD 步骤劣化严重，需人工引导命名才能获取准确结果。

## 关键概念
- SMILES: 分子结构的线性字符串表示法，用于化学信息学
- Img2Mol: 将分子结构图转换为 SMILES 表达式的深度学习模型
- CDDD 向量: 连续可微分子描述符，512 维分子嵌入表示
- NCI Cactus API: 美国国家癌症研究所提供的化学结构转换服务
- 取代基命名法: 通过系统命名描述复杂分子结构的化学命名方法

## 关联笔记
- 01KJBZGYAPQFV3CQ1A8H479FZ7.md: Img2Mol 论文阅读笔记，介绍将图片到 SMILES 任务拆解为图片到 CDDD 向量+CDDD 向量到 SMILES 的思路
- 01KJBZH3679KRGP1N7E91WZR91.md: Img2Mol 项目训练记录，包含训练过程和报错处理
- 01KJBZGZS514G9FB33XS2BKZ97.md: Img2Mol 项目测试记录，包含环境搭建和代码解读
