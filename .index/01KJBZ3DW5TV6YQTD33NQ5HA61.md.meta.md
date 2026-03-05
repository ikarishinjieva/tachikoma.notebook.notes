---
note: 01KJBZ3DW5TV6YQTD33NQ5HA61.md
title: 20230917 - 测试 codellama 微调 llama-recipes
indexed_at: 2026-03-05T09:04:03.276969+00:00
---

## 摘要
记录使用 llama-recipes 框架对 CodeLlama-7b-hf 进行 LoRA 微调的完整实验过程。包含训练命令配置、inference.py 改写、tokenizer 问题排查及自定义数据集微调验证。

## 关键概念
- llama-recipes: Meta 官方提供的 Llama 系列模型微调和部署工具库
- LoRA: 低秩适配器参数高效微调方法，仅训练少量参数
- PEFT: Parameter-Efficient Fine-Tuning，参数高效微调技术框架
- CodeLlama: Meta 发布的面向代码生成和理解的专用大语言模型

## 关联笔记
- 01KJBZ3BWM6KQH7FVEN8MHQN4F.md: 同一微调系列的后续实验，进行 SQLe 规则代码微调
- 01KJBZ3CDY4S7H5QCATKFHAPYS.md: 微调系列实验，跑通单元测试验证微调效果
- 01KJBZ3DN8NPCET2HBFXRWW19R.md: 微调系列实验，测试微调后模型的推广性
