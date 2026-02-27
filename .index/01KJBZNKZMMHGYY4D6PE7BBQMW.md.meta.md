---
note: 01KJBZNKZMMHGYY4D6PE7BBQMW.md
title: 20241209 - 使用ms-swift对Qwen2-VL进行微调 [12] - 调整loss函数
indexed_at: 2026-02-27T11:23:57.793730+00:00
---

用户要求我分析一篇关于使用 ms-swift 对 Qwen2-VL 进行微调并调整 loss 函数的笔记，生成索引元数据。

让我分析笔记内容：
1. 主题：对 Qwen2-VL 模型进行微调，用于识别分子结构图并输出 selfies 表达式
2. 核心问题：如何设计 loss 函数，对数字字段使用 MSE，对文本字段使用交叉熵
3. 遇到的困难：argmax 和 tokenizer decode 操作不可微，无法传递梯度
4. 讨论的解决方案：Teacher Forcing、Label Smoothing、数值嵌入表示、回归头等

标签应该包括：Qwen2-VL、模型微调、loss 函数、分子结构、selfies

摘要需要概括核心内容。

关键概念：
- Qwen2-VL：多模态大语言模型
- LoRA：参数高效微调方法
- Teacher Forcing：训练时直接使用目标序列作为输入
- MSE/CrossEntropy：损失函数类型

关联笔记：从提供的笔记列表看，这些都是关于数据库、MySQL、ClickHouse 等内容的笔记，与这篇 AI/ML 微调的笔记没有明显关联，所以应该是"无"。## 标签
Qwen2-VL, 模型微调, loss 函数, 分子结构识别, selfies, LoRA

## 摘要
记录使用 ms-swift 对 Qwen2-VL 进行 LoRA 微调，目标是让模型识别分子结构图并输出包含原子信息的 JSON 结构。探讨了将数字字段使用 MSE loss、文本字段使用交叉熵 loss 的方案，但发现 argmax 和 tokenizer decode 操作不可微导致梯度无法传播的问题。

## 关键概念
- Qwen2-VL: 阿里达摩院开源的多模态大语言模型，支持图像和文本输入
- LoRA: 低秩适配器，一种参数高效的微调方法，只训练少量增量参数
- Teacher Forcing: 训练时直接使用目标 token 作为输入，避免不可微操作
- 梯度传播: 反向传播中梯度通过可微操作传递回模型参数的过程

## 关联笔记
无
