---
note: 01KJBZ391BKTMFGSSBDRWEWS5H.md
title: 20230830 - 随机SQL生成
indexed_at: 2026-03-05T08:58:57.790454+00:00
---

## 摘要
介绍使用 MariaDB 的 randgen 工具通过.yy 语法文件随机生成 SQL 的方法。演示了如何调整 optimizer.yy 文件生成恒等条件，用于 SQL audit 训练时通过 DEGENERATE 标签识别结果。

## 关键概念
- randgen: MariaDB 的随机 SQL 生成工具，用于生成测试 SQL
- .yy 语法文件：定义 SQL 生成规则的语法描述文件
- 恒等条件：形如 tinyint = tinyint 的恒等式 WHERE 条件，用于训练识别
- DEGENERATE 标签：用于标识特殊生成规则的标记，便于审计时判断结果

## 关联笔记
- 01KJBZ3AKB9P49TFR3CP9PB0EQ.md: 直接使用 randgen 生成训练 SQL 进行 chatglm 微调
- 01KJBZ22JR5ERJF0NH6TAAP190.md: SQL 测试数据集的收集，与随机 SQL 生成目的相关
