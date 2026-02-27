---
note: 01KJBZ23YC0VMQ76EFC97D2ZJX.md
title: 20230723 - chatglm 微调, 对sql进行规则审核 [3]
indexed_at: 2026-02-27T10:33:26.990642+00:00
---

用户要求我分析这篇关于 ChatGLM 微调和 SQL 规则审核的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：ChatGLM-6B 模型微调，用于 SQL 规则审核
2. 主要内容：
   - 从 p-tuning 换回 LoRA
   - 问题：AST（抽象语法树）解析错误
   - 具体案例：limit 1 的 AST 生成错误，order by + limit + 子查询的 SQL 解析问题
   - 模型判断逻辑有问题

3. 关键概念：
   - ChatGLM-6B（大语言模型）
   - LoRA（微调方法）
   - AST（抽象语法树）
   - SQL 规则审核

4. 关联笔记分析：
   查看其他笔记列表，主要是 MySQL、Clickhouse、数据库相关的笔记。这篇笔记是关于 SQL 规则审核和 AI 模型微调的，与以下笔记可能有关联：
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 无明显关联
   - 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习 - 可能与 SQL 分析相关
   - 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 20210606 - 从 binlog 中分离 SQL, 进行统计 - 与 SQL 处理相关
   
   但整体来看，这篇笔记主要关注 AI 模型微调和 SQL AST 解析，与其他笔记的关联性不强。

现在生成索引元数据。## 标签
ChatGLM, LoRA, SQL 规则审核, AST 解析, 模型微调, 大语言模型

## 摘要
记录将 ChatGLM-6B 从 p-tuning 换回 LoRA 进行 SQL 规则审核微调的过程。核心问题是 AST 解析错误，尤其是含 order by + limit + 子查询的 SQL 语句，导致模型判断结论错误。

## 关键概念
- LoRA: 一种参数高效的微调方法，替代 p-tuning 用于模型训练
- AST: 抽象语法树，SQL 解析后的结构化表示，用于规则匹配
- SQL 规则审核: 通过模型分析 SQL 是否符合预定义的安全或规范规则

## 关联笔记
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 同为 SQL 分析工具学习，SQLLancer 也涉及 SQL 解析与审核
