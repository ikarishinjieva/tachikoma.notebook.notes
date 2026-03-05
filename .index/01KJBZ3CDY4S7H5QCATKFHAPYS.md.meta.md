---
note: 01KJBZ3CDY4S7H5QCATKFHAPYS.md
title: 20230923 - 测试用codellama, 进行SQLe规则代码的微调 - 跑通单元测试
indexed_at: 2026-03-05T09:03:18.697885+00:00
---

## 摘要
记录使用 CodeLlama-13b 微调生成 SQL 规则代码的实验经验，发现英文/中文描述、函数名三者均影响代码生成效果。提出增加同义描述增殖训练数据可显著提升效果，同时指出 MySQL 和 PG 在代码形式、单元测试签名、规则结构体等方面的差异问题。

## 关键概念
- 描述增殖：对同一 case 使用多种同义的中英文描述生成多个训练样本，提升训练效果
- 规则结构体：包含 Name、Desc、Annotation、Level、Category 等字段的规则定义结构
- 辅助函数签名：MySQL 和 PG 的单元测试辅助函数签名不同，影响模型泛化能力
- 字段类型检查：varchar/char 等字段类型的规则会相互影响，描述需准确

## 关联笔记
- 01KJBZ3DW5TV6YQTD33NQ5HA61.md: 20230917 测试 CodeLlama 微调 llama-recipes 的前置环境配置实验
- 01KJBZ3BWM6KQH7FVEN8MHQN4F.md: 20230920 使用 SQLe 的 DDL 规则代码进行微调的初步尝试
- 01KJBZ3DN8NPCET2HBFXRWW19R.md: 20231007 测试微调后模型在不同数据库间的规则推广能力
