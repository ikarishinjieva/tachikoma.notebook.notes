---
note: 01KJBYDAZJTQDJ7X3BQ52XVSSR.md
title: 20210525 - 临时磁盘表, INFORMATION_SCHEMA.FILES 和 `performance_schema`.`file_summary_by_instance` 观测不一致
indexed_at: 2026-03-05T07:25:50.986770+00:00
---

## 摘要
记录通过 INFORMATION_SCHEMA.FILES 和 performance_schema.file_summary_by_instance 两种方式观测 InnoDB 临时表空间 (ibtmp1) 时发现的数据不一致问题。通过实验对比两种观测方式的差异，分析 DATA_FREE 字段的来源机制。

## 关键概念
- INFORMATION_SCHEMA.FILES: MySQL 元数据表，提供表空间文件信息，包括 DATA_FREE 空闲空间
- performance_schema.file_summary_by_instance: 性能模式表，提供文件 I/O 读写统计信息
- innodb_temporary: InnoDB 临时表空间，用于存储内部临时表和用户临时表
- DATA_FREE: 表空间空闲空间大小，由 fsp_get_available_space_in_free_extents 函数计算
- fsp_get_available_space_in_free_extents: InnoDB 内部函数，对 space->free_len 进行部分调整

## 关联笔记
- 01KJBZB3ESK010EVRSR0XJDJFA.md: 涉及 MySQL 临时表满的排查思路，包含 innodb_temp_data_file_path 参数配置和临时表空间问题
- 01KJBYJ6VFVAX9WQ0HD1G3TPEM.md: 涉及 innodb_temporary 表空间的刷盘问题，SYS_TABLES 系统表应刷盘到临时表空间
