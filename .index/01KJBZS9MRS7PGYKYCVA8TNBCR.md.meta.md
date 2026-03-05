---
note: 01KJBZS9MRS7PGYKYCVA8TNBCR.md
title: 20250328 - 研究SQL优化模型微调 - 处理错误case[2]
indexed_at: 2026-03-05T11:36:59.558381+00:00
---

## 标签
SQL 优化，模型微调，投影下推，Qwen2.5-7B，训练数据分析，结构流程

## 摘要
记录使用 Qwen2.5-7B-Instruct 模型进行 SQL 优化微调的实验过程，采用"结构流程 + 改写指令"两步法替代原有三步法。测试多种训练参数配置（epoch 4/8/16、gradient_accumulation_steps），发现常见错误包括 v2 表误改写、第二层分析缺失等，尝试将结构分析改为从外到内广度搜索以改善遗漏问题。

## 关键概念
- 投影下推: SQL 优化规则，通过增加或删除无用列来优化查询结构
- 结构流程: 模型输出的 SQL 逻辑层次分析，用于指导改写指令生成
- 改写指令: 基于结构分析生成的具体 SQL 修改指示，模型需遵循执行
- 梯度累积步数 (gradient_accumulation_steps): 训练参数，影响批次大小和收敛效果

## 关联笔记
- 01KJBZSCARJVG6RJPDTR56Z3C2.md: 后续使用 DPO 让模型遵循改写指令，解决本笔记中发现的指令与 SQL 不一致问题
- 01KJBZSH34T3S96WRRSDA6EVVJ.md: 基于本笔记成果，使用两阶段验证稳定 SQL 修改的指令遵循效果后，重新梳理数据构建过程
- 01KJBZZJ92R5GJ0VH1J4W3X93K.md: 后期采用 GRPO 训练方法，重新制作数据生成流程以避免 GT 错误和匹配问题
