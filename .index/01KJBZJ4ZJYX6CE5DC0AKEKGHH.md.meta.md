---
note: 01KJBZJ4ZJYX6CE5DC0AKEKGHH.md
title: 20241029 - 在新gpu服务器上, 进行模型训练 (化学分子式识别)
indexed_at: 2026-03-05T10:53:37.028539+00:00
---

## 摘要
比较单节点、默认分布式、accelerate、deepspeed 四种分布式训练方式在 Qwen2-VL-7B 化学分子式识别任务上的性能差异。结论是几种方式训练时间和显存占用差异不大，最终选用默认分布式方案。

## 关键概念
- 默认分布式: PyTorch 原生 DDP，训练时间约 2:42:30，每显卡显存占用 18557MiB
- accelerate: HuggingFace 分布式训练工具，性能与默认分布式相当
- deepspeed: 微软深度学习优化库，训练时间略快 (2:33:34)，显存占用略低
- LoRA: 低秩适配器微调方式，rank=8, alpha=16，用于高效微调大模型

## 关联笔记
- 01KJBZJG2DN96WB9H43JKKN8MY.md: 同一训练任务的前期尝试，探索使用 CoT 进行分子式识别
- 01KJBZJSM5YEKX18QPBPFMAYNX.md: 同一训练任务的后续评估，分析过拟合原因
- 01KJBZHPBDM15T9Y2VCFNHP5TY.md: 数据集 decimer.ai 的前期调研和测试
