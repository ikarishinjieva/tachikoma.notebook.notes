---
note: 01KJBZJYMDWQXWHPAVTRVWV50H.md
title: 20241114 - 使用ms-swift对Qwen2-VL进行微调 [4] - 修改loss
indexed_at: 2026-02-27T11:21:58.066943+00:00
---

用户要求我分析一篇关于使用 ms-swift 对 Qwen2-VL 进行微调时修改 loss 的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：使用 ms-swift 框架对 Qwen2-VL 视觉语言模型进行 LoRA 微调
2. **核心问题**：在使用 loss_scale_config 时遇到 mask 形状不匹配的报错
3. **原因分析**：loss_scale 计算的 scale 数组长度与增长后的 labels 长度不匹配
4. **解决方案**：绕过方式（具体方案在笔记中似乎被截断了）
5. **训练效果**：v9 版本与 v8 相比 loss 整体抬升

现在生成元数据：

**标签**：需要 3-6 个关键词
- Qwen2-VL
- ms-swift
- 模型微调
- loss_scale
- LoRA
- 报错排查

**摘要**：2-3 句话
记录使用 ms-swift 对 Qwen2-VL 进行 LoRA 微调时遇到的 loss_scale 配置问题。分析了 mask 形状与 labels 长度不匹配的报错原因，并对比了 v9 与 v8 版本的训练效果。

**关键概念**：3-5 个
- Qwen2-VL: 阿里巴巴开源的视觉语言多模态模型
- ms-swift: 魔搭社区提供的模型微调框架


- loss_scale: 用于调整不同 token 在 loss 计算中的权重配置
- LoRA: 低秩适配器微调方法，参数高效的微调技术

**关联笔记**：
查看提供的笔记列表，都是 2021 年关于数据库、系统性能的内容，与这篇 2024 年 AI 模型微调的笔记没有关联。

所以关联笔记应该是「无」。
## 标签
Qwen2-VL, ms-swift, 模型微调, loss_scale, LoRA, 报错排查

## 摘要
记录使用 ms-swift 对 Qwen2-VL 进行 LoRA 微调时配置 loss_scale 遇到的 mask 形状不匹配报错。分析了输入中特殊 token 导致 labels 长度增长与 scale 数组长度不一致的原因，并对比了 v9 与 v8 版本的训练效果。

## 关键概念
- Qwen2-VL: 阿里巴巴开源的视觉语言多模态大模型
- ms-swift: 魔搭社区提供的模型微调框架
- loss_scale: 用于调整不同 token 在 loss 计算中权重的配置机制
- LoRA: 低秩适配器微调方法，参数高效的微调技术

## 关联笔记
无
