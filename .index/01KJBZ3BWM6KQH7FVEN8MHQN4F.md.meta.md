---
note: 01KJBZ3BWM6KQH7FVEN8MHQN4F.md
title: 20230920 - 测试用codellama, 进行SQLe规则代码的微调
indexed_at: 2026-02-27T10:35:11.589144+00:00
---

用户要求我分析这篇关于使用 SQLe 的 DDL 规则代码进行微调的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：使用 CodeLlama 对 SQLe 的 DDL 规则代码进行微调
2. **核心内容**：
   - 微调参数（epoch=10, loss=0.1294, temperature=0.1）
   - 多个 DDL 规则检查函数的代码示例
   - 尝试增加上下文和类型定义来改进结果

3. **标签**：应该包括 CodeLlama、微调、SQLe、DDL、规则检查、代码生成等

4. **摘要**：记录使用 CodeLlama 对 SQLe DDL 规则代码进行微调的过程，包括训练参数和多条规则代码的生成结果，以及通过增加上下文改进输出的尝试。

5. **关键概念**：
   - CodeLlama：用于代码生成的语言模型
   - SQLe：SQL 规则检查工具
   - DDL：数据定义语言
   - 微调：对预训练模型进行特定任务训练
   - AlterTableStmt：MySQL AST 中的 Alter Table 语句节点

6. **关联笔记**：查看其他笔记列表，寻找相关的：
   - 01KJBYDKM74PTS0GRF808YHQR7.md: 20210628 - DDL 过程梳理 - 与 DDL 相关
   - 其他笔记主要涉及 MySQL、Clickhouse 等，但与 DDL 规则代码微调关联较弱

我应该输出简洁的索引元数据。
## 标签
CodeLlama, 模型微调, SQLe, DDL 规则, 代码生成, AST

## 摘要
记录使用 CodeLlama 对 SQLe 的 DDL 规则检查代码进行微调的过程，包含训练参数和多条规则函数的生成结果。尝试通过增加类型定义上下文来改进代码生成质量。

## 关键概念
- CodeLlama: 用于代码生成的预训练语言模型
- SQLe: SQL 规则检查工具，提供 DDL/DML 规则校验
- AlterTableStmt: MySQL AST 中表示 ALTER TABLE 语句的节点类型
- 微调 (Fine-tuning): 使用特定领域数据对预训练模型进行适配训练
- temperature: 控制代码生成随机性的参数，值越低输出越确定

## 关联笔记
- 01KJBYDKM74PTS0GRF808YHQR7.md: 20210628 - DDL 过程梳理（同属 DDL 相关主题）
