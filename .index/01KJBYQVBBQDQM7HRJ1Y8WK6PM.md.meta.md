---
note: 01KJBYQVBBQDQM7HRJ1Y8WK6PM.md
title: 20221025 - 百胜, 唯一键表会有重复记录
indexed_at: 2026-03-05T08:12:49.389090+00:00
---

## 标签
MySQL, InnoDB, 唯一键冲突, 无主键表, innodb_space, 表空间分析

## 摘要
生产环境唯一键表出现重复记录，通过复制表空间文件到测试环境可复现，但 mysqldump 导入会报唯一键冲突。使用 innodb_space 分析发现无主键表会导致工具报错，暗示问题与无主键表的内部行 ID 机制相关。

## 关键概念
- innodb_space: MySQL InnoDB 表空间分析工具，用于检查表空间文件内部结构
- 无主键表: InnoDB 会自动生成隐藏的 row_id 作为聚簇索引
- 唯一键冲突: 违反唯一约束时产生的错误，通常表示存在重复数据

## 关联笔记
- 01KJBZAGWAND5W9R05TTT38871.md: 无主键表添加自增主键导致主从数据不一致和唯一键冲突的问题
