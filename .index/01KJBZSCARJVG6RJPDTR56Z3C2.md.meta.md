---
note: 01KJBZSCARJVG6RJPDTR56Z3C2.md
title: 20250407 - 研究SQL优化模型微调 - 让模型对SQL的修改 遵循 改写指令
indexed_at: 2026-03-05T11:37:27.940308+00:00
---

## 摘要
测试 Qwen2.5-7B-Instruct-1M 模型在 SQL 改写任务上的表现，验证模型能否遵循改写指令正确修改 SQL。通过逐步增加改写指令数量（1-4 个），评估 bleu-1<0.85 的错误率变化，发现指令越多错误率越高。

## 关键概念
- 改写指令: 要求模型根据分析流程对 SQL 进行改写的提示指令
- bleu-1: 评估 SQL 改写准确性的指标，阈值设为 0.85
- BIRD-SQL-data-train: 用于生成评测数据的 SQL 数据集
- swift eval: 模型评估命令工具

## 关联笔记
- 01KJBZS9MRS7PGYKYCVA8TNBCR.md: 前继笔记，处理错误 case[2] 的研究
- 01KJBZSH34T3S96WRRSDA6EVVJ.md: 后续笔记，重新梳理 SQL 优化的数据构建过程
- 01KJBZS8FA11EJ57E9H7M1V5WE.md: 更早的前继，处理错误 case 的初始研究
