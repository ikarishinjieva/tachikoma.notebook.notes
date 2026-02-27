---
note: 01KJBZNZQ5X5K7KZJPBRPS7VTZ.md
title: 20241216 - 使用ms-swift对Qwen2-VL进行微调 [13] - 递增式训练
indexed_at: 2026-02-27T11:24:19.769709+00:00
---

用户要求我分析这篇关于 Qwen2-VL 微调的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20241216 - 使用 ms-swift 对 Qwen2-VL 进行微调 [13] - 递增式训练
2. 内容主要是：
   - 一个三阶段递增式训练脚本（phase-1, phase-2, phase-3）
   - 使用 ms-swift 框架对 Qwen2-VL-7B-Instruct 进行 LoRA 微调
   - 数据集按长度递增（length_1_4, length_5, length_6）
   - 效果评估和结果分析

3. 标签应该包括：Qwen2-VL、LoRA 微调、递增式训练、ms-swift、多模态等

4. 摘要：描述三阶段递增训练流程和效果

5. 关键概念：LoRA、递增式训练、Qwen2-VL、ms-swift、checkpoint merge 等

6. 关联笔记：查看其他笔记列表，都是 2021 年的数据库/系统相关笔记，与这篇 2024 年的 AI 微调笔记没有明显关联

让我生成输出。## 标签
Qwen2-VL, LoRA 微调，递增式训练，ms-swift, 多模态大模型

## 摘要
记录使用 ms-swift 框架对 Qwen2-VL-7B 进行三阶段递增式 LoRA 微调的完整流程，按数据长度分阶段训练并合并 checkpoint。最终效果较差，产生格式违法输出，正在探索原因。

## 关键概念
- LoRA: 低秩适配器微调方法，只训练少量参数
- 递增式训练：按数据复杂度分阶段逐步训练的策略
- ms-swift: 魔搭社区的大模型微调框架
- checkpoint merge: 将 LoRA 权重合并到基座模型的操作

## 关联笔记
无
