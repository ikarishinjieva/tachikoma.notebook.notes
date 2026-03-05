---
note: 01KJBZSH9AJT9PM0QF3R1R4Y35.md
title: 20250506 - 阅读论文*: SQL-R1: Training Natural Language to SQL Reasoning Model By Reinforcement Learning
indexed_at: 2026-03-05T11:38:17.755251+00:00
---

## 摘要
SQL-R1 论文提出将强化学习引入 Text2SQL 任务，采用先 SFT 后 RL 的冷启动训练方式。核心创新是设计了包含格式、执行、结果、长度四种奖励的复合渐进式反馈机制，并配合自洽性投票策略提升 SQL 生成准确性。

## 关键概念
- 冷启动训练: 先进行监督微调 (SFT)，再进行强化学习 (RL) 的两阶段训练方法
- 复合奖励函数: 由格式奖励、执行奖励、结果奖励、长度奖励组成的渐进式反馈机制
- 自洽性投票: 使用高温度生成多组 SQL，选取多次出现的答案作为最终输出
- 格式奖励: 检查推理过程和 SQL 答案是否符合预设标签格式规范
- 执行奖励: 评估生成的 SQL 候选是否可执行，防止语法错误

## 关联笔记
- 01KJBZRVT5F0HWQY45ZK497JMA.md: SQL-o1 论文笔记，同样研究 Text2SQL 领域的强化学习方法，使用 MCTS 搜索和自奖励机制
- 01KJBZPTT8A443T3Q5V8SWX4QJ.md: KeyInst 论文笔记，讨论 Text2SQL 中的关键字指令和常见错误类型
- 01KJBZY39GJE38CB7BYSTRWMBX.md: SQL 优化 GRPO 训练实践笔记，记录强化学习奖励函数设计和训练调优经验
