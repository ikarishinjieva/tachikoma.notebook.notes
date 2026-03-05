---
note: 01KJBZKTQRNA64HW30T7QG691S.md
title: 20241119 - 使用ms-swift对Qwen2-VL进行微调 [6] - 增强Selfies的效果
indexed_at: 2026-03-05T11:00:30.584134+00:00
---

## 摘要
尝试通过修改loss函数同时训练SELFIES表达式生成和合法性检查，对非法表达式部分进行惩罚。实验结果显示改造loss后eval/loss反而更高，未能解决过拟合问题。

## 关键概念
- SELFIES: 一种比SMILES更鲁棒的分子表达式表示方法
- 多任务loss: 同时优化表达式生成和合法性验证的复合loss函数
- 特殊标记扩展: 通过添加[VALID]/[INVALID]等标记让模型学习合法性判断

## 关联笔记
- 01KJBZKJR3KRVFZRXR50RG57YM.md: 前序笔记，记录过拟合问题分析和本实验的动机
- 01KJBZJYMDWQXWHPAVTRVWV50H.md: 相关实验，之前尝试修改loss权重的记录
- 01KJBZKX971VTKSDKYRNX3ZV7N.md: 后续笔记，转向使用规范数据减少过拟合的方案
