---
note: 01KJBZ3DN8NPCET2HBFXRWW19R.md
title: 20231007 - 测试codellama微调后的推广性
indexed_at: 2026-03-05T09:03:48.549585+00:00
---

## 标签
CodeLlama, 微调, 规则推广, MySQL DDL, PostgreSQL, 代码生成

## 摘要
测试 CodeLlama 微调后将 PostgreSQL DDL 规则代码推广到 MySQL 的效果。包含规则检查函数实现和单元测试用例，发现生成的代码存在辅助函数逻辑缺失问题。

## 关键概念
- 规则推广：将已训练的规则从一种数据库方言迁移到另一种方言
- RuleHandlerInput：规则处理器的输入结构，包含 AST 节点和参数
- DDL 检查：对数据定义语言语句进行语法和语义验证
- AST 解析：使用抽象语法树分析 SQL 语句结构

## 关联笔记
- 01KJBZ3DW5TV6YQTD33NQ5HA61.md: CodeLlama 微调 llama-recipes 的初步测试
- 01KJBZ3BWM6KQH7FVEN8MHQN4F.md: CodeLlama 微调 SQLe 规则代码的早期实验
- 01KJBZ3CDY4S7H5QCATKFHAPYS.md: CodeLlama 微调跑通单元测试的前置验证
