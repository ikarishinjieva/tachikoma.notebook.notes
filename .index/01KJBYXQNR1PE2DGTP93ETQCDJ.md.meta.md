---
note: 01KJBYXQNR1PE2DGTP93ETQCDJ.md
title: 20230118 - 中移 crash
indexed_at: 2026-02-27T10:23:55.702978+00:00
---

用户要求我分析一篇关于 MySQL 崩溃的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **标题**：20230118 - 中移 crash
2. **内容**：MySQL 8.0.30 升级后崩溃，通过 debug 版本生成 coredump 分析，堆栈显示是 InnoDB 的 record 转换问题（rec_convert_dtuple_to_rec_new），涉及版本化记录（rec_new_is_versioned）

**标签**应该是：MySQL, InnoDB, 崩溃分析, coredump, 升级问题

**摘要**：记录 MySQL 8.0.30 从 8.0.27 升级后运行一段时间崩溃的问题。通过 mysql-debug 生成 coredump 分析，定位到 InnoDB 的 rec_new_is_versioned 断言失败。

**关键概念**：
- coredump: 程序崩溃时生成的内存转储文件，用于事后分析
- InnoDB: MySQL 的存储引擎
- rec_new_is_versioned: InnoDB 中检查记录是否版本化的断言
- btr_cur: B+ 树游标，用于 InnoDB 索引操作
- row_undo_mod: 行级更新回滚操作

**关联笔记**：查看提供的笔记列表，寻找与 MySQL、InnoDB、崩溃分析相关的：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理（InnoDB 相关）


- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究（MySQL 相关）
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿（MySQL 相关）
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: 20210628 - Update 是如何进行的（与堆栈中的 row_undo_mod 相关）

这些笔记都涉及 MySQL 内部机制，特别是 InnoDB 存储引擎的操作流程，与当前崩溃问题的分析方向相符。## 标签
MySQL, InnoDB, 崩溃分析, coredump, 版本升级, 断言失败

## 摘要
记录 MySQL 8.0.30 从 8.0.27 升级后运行一段时间崩溃的问题。通过 mysql-debug 生成 coredump 分析，定位到 InnoDB 的 `rec_new_is_versioned` 断言失败，堆栈涉及记录转换和更新回滚操作。

## 关键概念
- coredump: 程序崩溃时生成的内存转储文件，用于事后调试分析
- rec_new_is_versioned: InnoDB 中检查记录是否为版本化记录的断言函数
- btr_cur: B+ 树游标，用于 InnoDB 索引的遍历和修改操作
- row_undo_mod: InnoDB 事务回滚时执行行级更新撤销的操作
- dtuple_to_rec: 将数据元组转换为物理记录格式的内部函数

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 涉及 InnoDB 刷脏页机制，与 InnoDB 内部操作相关
- 01KJBYDMR61HHS1TC4FYJG6YJ5.md: 讲解 Update 操作过程，与堆栈中的 row_undo_mod 相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究，同属 InnoDB 内部机制分析
