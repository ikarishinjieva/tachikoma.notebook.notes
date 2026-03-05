---
note: 01KJBZN4KQV7NK5GRT46165QTR.md
title: 20241203 - 模型微调知识整理
indexed_at: 2026-03-05T11:03:48.025977+00:00
---

## 摘要
整理大模型微调的基础知识与三次实践尝试：ChatGLM 用于 SQL 审核、CodeLlama 生成审核代码、ms-swift 微调多模大模型。总结关键参数（lora_rank、LR、epoch）、过拟合解决方案及工具选择建议。

## 关键概念
- LoRA: 低秩适配技术，通过 rank 和可训练层数控制模型复杂度
- 过拟合: 训练集表现好但校验集差，需降低复杂度并增强数据
- 校验集: 独立于训练集，用于选择最佳模型而非参与训练
- ms-swift: 推荐的多模态大模型微调工具，替代存在 bug 的 Llama-factory

## 关联笔记
- 01KJBZJT9YFKP1XPAKC0PHG4GA.md: 20241112 日 Qwen2-VL 微调的起点笔记，文中多次引用其多模态模型结构分析
- 01KJBZKX971VTKSDKYRNX3ZV7N.md: 20241121 日用规范数据减少过拟合的实践，对应笔记中数据增强有效性结论
- 01KJBZ1AAKZQX8R8K00CQV62YD.md: 20230616 日 ChatGLM 微调用于 SQL 审核的最初尝试，标记为边界数字不敏感
