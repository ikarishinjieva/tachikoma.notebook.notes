---
note: 01KJBZH3679KRGP1N7E91WZR91.md
title: 20241018 - 训练Img2Mol项目
indexed_at: 2026-02-27T11:18:55.585154+00:00
---

用户要求我分析一篇关于训练 Img2Mol 项目的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20241018 - 训练 Img2Mol 项目
2. 主要内容：
   - 训练脚本 train.ipynb
   - 遇到的报错：AttributeError related to tensorflow.io.gfile
   - 修复方法：修改 tensorboard 代码
   - model.validation_step 中的维度问题
   - 用 tensorboard 查看结果，需要新建环境
   - 查看预测结果

这是一篇关于深度学习/机器学习项目训练过程的笔记，涉及：
- Img2Mol（图像到分子结构）
- PyTorch Lightning
- TensorBoard
- 环境配置和报错解决

现在需要生成：
1. 标签（3-6 个）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

看其他笔记列表，都是关于数据库、MySQL、ClickHouse、系统性能等方面的，与这篇深度学习/化学信息学的笔记没有明显关联。

让我生成索引：

标签：Img2Mol, 深度学习，训练调试，TensorBoard, PyTorch Lightning, 环境配置

摘要：记录训练 Img2Mol 模型过程中遇到的报错及解决方案，包括 TensorBoard 兼容性问题。

涉及模型验证维度不匹配和 TensorBoard 环境配置等具体技术细节。

关键概念：
- Img2Mol: 从图像预测分子结构的深度学习模型
- PyTorch Lightning: 基于 PyTorch 的深度学习框架
- TensorBoard: 深度学习可视化工具
- CDDD: 连续离散分子描述符（分子表示方法）

关联笔记：无（其他笔记都是数据库/系统相关，与深度学习无关）## 标签
Img2Mol, 深度学习，训练调试，TensorBoard, PyTorch Lightning, 环境配置

## 摘要
记录训练 Img2Mol 模型过程中遇到的报错及解决方案，包括 TensorBoard 与 TensorFlow 版本兼容性问题。涉及模型验证阶段维度不匹配修复和 TensorBoard 环境独立配置。

## 关键概念
- Img2Mol: 从化学结构图像预测分子表示的深度学习模型
- PyTorch Lightning: 基于 PyTorch 的高层训练框架
- TensorBoard: 深度学习训练过程可视化工具
- CDDD: 连续离散分子描述符，用于分子表示学习

## 关联笔记
无
