---
note: 01KJBYXVP2BC1JDG5FZX6SMMF0.md
title: 20230412 - 测试 alpaca 与 llama_index结合
indexed_at: 2026-03-05T08:34:52.905854+00:00
---

## 摘要
记录使用 alpaca-lora 模型与 llama_index 结合的测试过程，尝试 7b/13b/30b 不同规模模型的效果对比。13b 模型效果优于 7b，但面对模糊问题仍不理想；30b 模型因显存不足切换至 CPU 运行，速度慢且效果一般。

## 关键概念
- PEFT (Parameter-Efficient Fine-Tuning): 对预训练模型仅微调少数参数的技术
- LoRA: 低秩适应微调方法，可大幅减少可训练参数量
- load_in_8bit: 8 比特量化加载模型，减少显存占用但需 bitsandbytes 支持

## 关联笔记
- 01KJBYY15XHA2SXQJ1QNJY255A.md: 同系列 vicuna-13b + llama_index 测试，同样无法识别 enlarge 命令
- 01KJBYYB3AQZY7FMQ8AHZGV7MY.md: 同系列 koala-13b + llama_index 测试，效果对比参考
- 01KJBYXSSKV6FMJ3Q1WQ6M5EYZ.md: 早期 alpaca 与 llama-index 联用的技术线索收集
