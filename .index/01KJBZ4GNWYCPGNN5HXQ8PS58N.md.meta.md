---
note: 01KJBZ4GNWYCPGNN5HXQ8PS58N.md
title: 20231218 - 在Oceanbase上, 开发MySQL的audit log
indexed_at: 2026-02-27T10:41:22.327247+00:00
---

用户希望我分析这篇关于 OceanBase 上开发 MySQL audit log 的笔记，生成索引元数据。让我仔细分析笔记内容：

1. **主题**：在 OceanBase 数据库上开发 MySQL 的 audit log 功能
2. **核心内容**：
   - 寻找可模仿的内部表结构
   - 详细分析了 `__all_ddl_operation` 表的结构定义
   - 包括租户表结构初始化、索引初始化、全局表结构初始化
   - 表结构代码定义和调用
   - 插入记录的代码逻辑

3. **关键概念**：
   - OceanBase 内部表（__all_ddl_operation 等）
   - 租户表结构
   - audit log（审计日志）
   - DDL operation
   - 系统表定义

4. **关联笔记分析**：
   - 查看其他笔记列表，寻找与 MySQL、DDL、audit、performance_schema 相关的
   - `01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema的日常使用` - 与 MySQL 监控/审计相关
   - `01KJBYDKM74PTS0GRF808YHQR7.md: 20210628 - DDL 过程梳理` - 与 DDL 相关
   - 其他笔记大多是 Clickhouse、内存、刷盘等主题，关联度不高

现在生成索引元数据：
## 标签
OceanBase, MySQL, audit log, DDL, 内部表, 审计日志

## 摘要
记录在 OceanBase 数据库上开发 MySQL audit log 功能时，对内部表结构的调研过程。重点分析了 `__all_ddl_operation` 等系统表的表结构定义、索引初始化和代码实现。

## 关键概念
- __all_ddl_operation: OceanBase 记录 DDL 操作的系统内部表
- 租户表结构: OceanBase 中每个租户独立的表空间定义
- GV$OB_SQL_AUDIT: OceanBase 全局 SQL 审计视图
- def_table_schema: OceanBase 内部表结构定义的宏函数

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 都涉及 MySQL 审计/监控相关的 performance_schema
- 01KJBYDKM74PTS0GRF808YHQR7.md: 都涉及 DDL 操作的处理流程
