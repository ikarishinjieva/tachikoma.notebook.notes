---
note: 01KJBYDAZJTQDJ7X3BQ52XVSSR.md
title: 20210525 - 临时磁盘表, INFORMATION_SCHEMA.FILES 和 `performance_schema`.`file_summary_by_instance` 观测不一致
indexed_at: 2026-02-27T10:02:51.473339+00:00
---

用户要求我分析一篇关于 MySQL 临时磁盘表观测不一致的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：INFORMATION_SCHEMA.FILES 和 performance_schema.file_summary_by_instance 在观测临时表空间时数据不一致
2. 涉及内容：
   - 造表实验
   - 两种观测方式的对比
   - data_free 的来源分析
   - 性能统计数据的对比

关键概念：
- INFORMATION_SCHEMA.FILES
- performance_schema.file_summary_by_instance
- InnoDB 临时表空间 (innodb_temporary)
- ibtmp1
- DATA_FREE

关联笔记分析：
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - 直接相关，都涉及 performance_schema
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 可能相关，涉及资源观测
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页 - 涉及 InnoDB 刷页机制

现在生成索引元数据。
## 标签
MySQL, InnoDB, 临时表空间, performance_schema, INFORMATION_SCHEMA, 观测分析

## 摘要
记录通过 INFORMATION_SCHEMA.FILES 和 performance_schema.file_summary_by_instance 两种方式观测 InnoDB 临时表空间 (ibtmp1) 时数据不一致的现象。分析 data_free 的来源并设计实验进行对比验证。

## 关键概念
- INFORMATION_SCHEMA.FILES: 提供表空间文件元数据信息，包括 DATA_FREE 空闲空间
- performance_schema.file_summary_by_instance: 按文件实例汇总的 I/O 统计信息
- innodb_temporary: InnoDB 临时表空间，用于存储临时表和磁盘临时表
- DATA_FREE: 表空间中已分配但未使用的空闲字节数
- ibtmp1: InnoDB 共享临时表空间数据文件

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - 同属 performance_schema 观测技术研究
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 同属 MySQL 资源观测与分析主题
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页 - 涉及 InnoDB 刷页机制研究
