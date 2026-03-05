---
note: 01KJBZDYTJF2N82HT70Z06VNQN.md
title: 20240811 - AutoPrompt架构
indexed_at: 2026-03-05T10:30:33.630820+00:00
---

## 标签
AutoPrompt, 提示词优化, LLM, 架构分析, 迭代优化, 对抗样本

## 摘要
分析 AutoPrompt 项目的核心架构和优化流程，包括命令行入口、优化管道执行步骤、各组件（annotator/predictor/eval）的配置选项。记录了 step_prompt 和对抗样本生成的提示词模板及其翻译，并指出深度搜索策略和样本充足时不触发对抗数据生成的问题。

## 关键概念
- OptimizationPipeline: AutoPrompt 的核心优化管道，执行多轮提示词调优步骤
- step_prompt: 根据错误分析和历史得分生成新提示词的元提示词
- 对抗样本生成: 当样本不足时生成挑战性样本以增强模型鲁棒性
- 深度搜索策略: 持续向下搜索而不因评分下降后退的优化策略

## 关联笔记
- 01KJBZE545BYHRCYTWBJMQ6PJS.md: 20240812 试用 AutoPrompt 的实操记录和配置说明
- 01KJBZEB61ZC33F8KAY5CR9GDJ.md: 20240814 对 AutoPrompt 进行日志和流程分析的后续笔记
- 01KJBZAZR633GF3JXZFN590BXE.md: 提及其他提示词优化项目包括 AutoPrompt 的对比笔记
