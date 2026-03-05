---
note: 01KJBZC8W6D9K78AHWW3G1S047.md
title: 20240730 - 理解Dspy
indexed_at: 2026-03-05T10:28:11.520000+00:00
---

## 标签
DSPy, LLM 优化，提示词工程，优化器，Few-Shot, 模型蒸馏

## 摘要
系统解析了 DSPy 框架中的多种优化器，包括 BootstrapFewShot、COPRO、MIPROv2、KNNFewShot 等。详细说明了各优化器的工作原理、配置参数、适用场景及局限性。

## 关键概念
- Signature: LLM 调用的函数签名，用于生成提示词的输入输出参数
- Predictor: 对 LLM 调用的函数，通过 Signature 生成
- BootstrapFewShot: 使用 teacher 模型生成增强 demo 作为 few-shot 示例的训练器
- COPRO: 组合式提示词排名优化，多轮迭代生成评分最高的提示词
- MIPROv2: 多指令、提示和排名优化，使用 Optuna 在高维空间搜索最优配置

## 关联笔记
- 01KJBZCHRA520E774GE1NMCPY3.md: 被本笔记引用，记录了 COPRO 和 MIPROv2 的详细实验过程
- 01KJBZDAD5PP3XK4H8WDZ9DSZA.md: 后续对 COPRO 优化器的改进实验
- 01KJBZD8WG7RH0DJMMACNG562Q.md: DSPy 在 ChatDBA 项目中的实际应用测试
