---
note: 01KJBZCHRA520E774GE1NMCPY3.md
title: 20240801 - dspy 实验
indexed_at: 2026-03-05T10:28:25.137335+00:00
---

## 标签
DSPy, 提示词优化, COPRO, MIPROv2, Optuna, 实验记录

## 摘要
记录 DSPy 框架中 COPRO 和 MIPROv2 两种提示词优化器的执行流程解析。分析了 COPRO 的多轮迭代生成机制和 MIPROv2 结合 Optuna 进行调优的特点及局限性。

## 关键概念
- COPRO: DSPy 的组合提示词排序优化器，通过多轮迭代生成和评分提示词候选
- MIPROv2: 多指令提示词优化器，使用 Optuna 在高维空间搜索最优提示词组合
- Optuna: 超参数优化框架，适合在高维空间进行搜索调优
- BootstrapFewShot: 通过自举生成 few-shot 样例集的提示词优化方法

## 关联笔记
- 01KJBZC8W6D9K78AHWW3G1S047.md: 详细解析 COPRO 和 MIPROv2 优化器的原理与代码实现
- 01KJBZDAD5PP3XK4H8WDZ9DSZA.md: 后续对 COPRO 进行改进的实验记录
- 01KJBZD8WG7RH0DJMMACNG562Q.md: 使用 DSPy 和 COPRO 测试 ChatDBA 的实际应用案例
