---
note: 01KJBZJT9YFKP1XPAKC0PHG4GA.md
title: 20241112 - 使用ms-swift对Qwen2-VL进行微调 [1]
indexed_at: 2026-02-27T11:21:35.557294+00:00
---

用户要求我分析一篇关于使用 ms-swift 对 Qwen2-VL 进行微调的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主要内容**：笔记记录了使用 ms-swift 框架对 Qwen2-VL-7B-Instruct 多模态模型进行 LoRA 微调的过程，包括环境配置、训练命令、以及调整超参数（weight_decay、lora_dropout、lora_rank）来解决过拟合问题的尝试。

2. **标签**：应该包含 ms-swift、Qwen2-VL、LoRA 微调、多模态、过拟合、深度学习等关键词

3. **摘要**：需要概括笔记的核心内容 - 记录使用 ms-swift 框架对 Qwen2-VL 进行 LoRA 微调的环境配置、训练命令，以及尝试调整超参数解决过拟合问题的过程。

4. **关键概念**：
   - ms-swift: ModelScope 提供的模型微调框架
   - Qwen2-VL: 阿里云通义千问多模态大语言模型
   - LoRA: 低秩适配器，一种参数高效的微调方法
   - DeepSpeed: 深度学习训练优化库
   - 过拟合：模型在训练集表现好但泛化能力差的现象

5. **关联笔记**：查看提供的其他笔记列表，都是关于数据库（MySQL、Clickhouse）、系统性能、网络等主题的，与这篇 AI/深度学习微调的笔记没有明显关联。

所以关联笔记应该是「无」。

现在我来生成输出。
## 标签
ms-swift, Qwen2-VL, LoRA 微调, 多模态大模型, 过拟合, 深度学习

## 摘要
记录使用 ms-swift 框架对 Qwen2-VL-7B-Instruct 多模态模型进行 LoRA 微调的完整流程，包括环境配置和训练命令。多次调整 weight_decay、lora_dropout、lora_rank 等超参数尝试解决过拟合问题，但训练曲线未见明显改善。

## 关键概念
- ms-swift: ModelScope 提供的统一模型微调框架，支持多种大模型的训练和推理
- Qwen2-VL: 阿里云通义千问系列的多模态大语言模型，支持图像和文本输入
- LoRA: 低秩适配器技术，通过训练少量参数实现大模型的高效微调
- DeepSpeed: 深度学习训练优化库，提供 ZeRO 等显存优化技术
- 过拟合: 模型在训练集上表现良好但泛化能力不足的现象

## 关联笔记
无
