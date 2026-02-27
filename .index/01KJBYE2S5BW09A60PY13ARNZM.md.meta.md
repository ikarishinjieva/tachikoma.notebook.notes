---
note: 01KJBYE2S5BW09A60PY13ARNZM.md
title: 20210730 - i_s.columns 排序不一致
indexed_at: 2026-02-27T10:08:00.767060+00:00
---

用户希望我分析这篇关于 MySQL information_schema.columns 排序不一致问题的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主题：MySQL 8.0.19 中 information_schema.COLUMNS 查询结果排序不一致的问题
2. 现象：不同配置（lower_case_table_names）和不同机器上，查询结果排序方式不同
3. 分析：涉及 InnoDB 内部表访问的堆栈分析

关键概念：
- information_schema.columns
- lower_case_table_names
- InnoDB 内部表
- ORDINAL_POSITION
- 堆栈分析

关联笔记：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 如何后台不刷脏页 - 涉及 InnoDB 内部机制
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 涉及 MySQL 系统表/视图
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: INFORMATION_SCHEMA.FILES 观测不一致 - 涉及 INFORMATION_SCHEMA 不一致问题

让我生成简洁的索引元数据。
## 标签
MySQL, information_schema, columns 排序, InnoDB, lower_case_table_names, 堆栈分析

## 摘要
记录 MySQL 8.0.19 中查询 information_schema.COLUMNS 时返回结果排序不一致的现象，不同机器和配置下排序行为存在差异。通过堆栈分析发现 i_s.columns 是视图，底层访问 mysql.columns 内部表时无显式排序语句。

## 关键概念
- information_schema.COLUMNS: MySQL 元数据视图，展示表列信息但无显式排序
- lower_case_table_names: 控制表名大小写敏感性的配置参数，影响排序行为
- ORDINAL_POSITION: 列在表中的定义顺序，期望的排序依据
- InnoDB 内部表: mysql.columns 是存储列元数据的 InnoDB 引擎内部表

## 关联笔记
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 同样涉及 INFORMATION_SCHEMA 视图观测数据不一致的问题
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 涉及 InnoDB 内部机制和刷脏页行为研究
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 涉及 MySQL performance_schema 等系统表的使用
