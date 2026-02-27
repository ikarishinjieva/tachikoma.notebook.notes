---
note: 01KJBZ3DN8NPCET2HBFXRWW19R.md
title: 20231007 - 测试codellama微调后的推广性
indexed_at: 2026-02-27T10:35:30.814138+00:00
---

用户要求我分析这篇笔记并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20231007 - 测试 codellama 微调后的推广性
2. 主要内容：测试 CodeLlama 微调模型在 SQL 规则检查方面的推广能力，从 PostgreSQL 规则推广到 MySQL
3. 涉及技术：CodeLlama 微调、MySQL DDL 规则检查、AST 解析、SQL 规则引擎

关键概念：
- CodeLlama 微调
- MySQL DDL 规则检查
- AST（抽象语法树）解析
- SQL 规则引擎

关联笔记：从列表中找相关的，主要看 MySQL、DDL、Clickhouse 等相关的
- 01KJBYDKM74PTS0GRF808YHQR7.md: 20210628 - DDL 过程梳理 - 都涉及 DDL 相关内容
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习 - 涉及 MySQL
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表，INFORMATION_SCHEMA.FILES... - 涉及 MySQL 相关

现在按照格式输出。
## 标签
CodeLlama, 微调测试, MySQL DDL, 规则引擎, AST 解析, SQL 规则检查

## 摘要
测试 CodeLlama 微调模型将 PostgreSQL 规则推广到 MySQL 的效果，验证 DDL 检查规则的生成能力。包含完整的 Go 代码实现和测试用例，展示规则从 PG 到 MySQL 的迁移过程。

## 关键概念
- CodeLlama 微调：针对 SQL 规则检查任务微调的大语言模型
- DDL 规则检查：验证建表语句是否符合规范的自动化检查
- AST 解析：通过抽象语法树分析 SQL 语句结构
- 规则推广性：将已训练的规则从一种数据库迁移到另一种数据库的能力

## 关联笔记
- 01KJBYDKM74PTS0GRF808YHQR7.md: 20210628 - DDL 过程梳理（都涉及 DDL 相关技术研究）
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习（涉及 MySQL 技术研究）
