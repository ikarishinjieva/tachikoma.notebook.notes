---
note: 01KJBZJMH94H8E9GBJBFTFCW58.md
title: 20241108 - 尝试输出transformers的evaluate结果
indexed_at: 2026-03-05T10:56:04.255892+00:00
---

## 标签
transformers, evaluate, predict_with_generate, Qwen2-VL, llama-factory, 分子式识别

## 摘要
调查 transformers 训练过程中的 evaluate 机制，研究 predict_with_generate 参数的评估方式差异。修改 llama-factory 代码解决参数检查错误和 Qwen2-VL 的 left-padded tensor 问题，成功开启基于生成序列的评估指标。

## 关键概念
- predict_with_generate: 决定评估方式是基于 token 预测概率还是基于生成序列的整体质量指标（如 BLEU/ROUGE）
- _maybe_log_save_evaluate: transformers trainer 中在训练过程中调用 evaluate 的函数
- 基于 token 的评估: 关注每个位置预测的准确性，计算交叉熵损失
- 基于生成序列的评估: 关注完整序列的质量，使用 BLEU/ROUGE 等指标

## 关联笔记
- 01KJBZJSM5YEKX18QPBPFMAYNX.md: 同样涉及 transformers trainer.py 的训练错误排查
- 01KJBZJ4ZJYX6CE5DC0AKEKGHH.md: Qwen2-VL 微调的早期实验配置记录
- 01KJBZNVMJEEW6VS461MEX1YH0.md: 分子式识别任务的阶段性移交，包含相关实验背景
