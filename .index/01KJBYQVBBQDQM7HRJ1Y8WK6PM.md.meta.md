---
note: 01KJBYQVBBQDQM7HRJ1Y8WK6PM.md
title: 20221025 - 百胜, 唯一键表会有重复记录
indexed_at: 2026-02-27T10:19:28.713451+00:00
---

用户希望我分析这篇关于 MySQL 唯一键表出现重复记录的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：百胜项目中，唯一键表出现重复记录的问题
2. 现象：生产表空间文件在测试环境可复现，但 mysqldump 导入会报唯一键冲突
3. 分析：使用 innodb_space 工具分析，发现无主键表会导致 innodb_space 报错

关键概念：
- 唯一键冲突
- innodb_space 工具
- 无主键表
- 表空间文件
- mysqldump

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理 - 涉及 InnoDB
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 - 涉及 MySQL
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表，INFORMATION_SCHEMA.FILES 和 performance_schema.file_summary_by_instance 观测不一致 - 涉及表空间/文件分析
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+学习笔记 - 时间接近但主题不同

最相关的是涉及 InnoDB、表空间分析的笔记。

现在生成索引元数据。
## 标签
MySQL, InnoDB, 唯一键冲突, innodb_space, 无主键表, 表空间文件

## 摘要
百胜项目中发现唯一键表存在重复记录异常，通过复制生产表空间文件在测试环境可复现，但 mysqldump 导入会报唯一键冲突。使用 innodb_space 工具分析时发现该表为无主键表，导致工具报错。

## 关键概念
- 唯一键冲突：表中唯一索引约束被违反，出现重复键值
- innodb_space：InnoDB 表空间文件分析工具，用于诊断底层存储问题
- 无主键表：没有定义主键的 InnoDB 表，内部使用隐藏的 ROW_ID 作为聚簇键
- 表空间文件：InnoDB 存储引擎的物理数据文件（.ibd），包含实际的表数据和索引

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理（涉及 InnoDB 刷页机制）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究（涉及 MySQL 脏页刷新研究）
