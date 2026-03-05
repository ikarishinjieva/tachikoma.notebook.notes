---
note: 01KJBZ4GNWYCPGNN5HXQ8PS58N.md
title: 20231218 - 在Oceanbase上, 开发MySQL的audit log
indexed_at: 2026-03-05T09:24:42.227290+00:00
---

## 标签
OceanBase, MySQL, audit log, DDL, 内部表, 表结构定义

## 摘要
记录在 OceanBase 上开发 MySQL audit log 功能时，对内部表结构的分析和实现方式。重点研究了__all_ddl_operation 等系统表的表结构定义、索引初始化和代码调用关系。

## 关键概念
- __all_ddl_operation: OceanBase 内部记录 DDL 操作的系统表
- ob_inner_table_schema_def.py: 定义 OceanBase 内部表结构的 Python 配置文件
- 租户表/全局表: OceanBase 中两种不同的表空间类型，分别用于租户级和全局级数据
- GV$OB_SQL_AUDIT: OceanBase 全局 SQL 审计视图

## 关联笔记
- 01KJBZ4J03FH868Z4ZDKGPQRDY.md: 同一系列后续笔记，继续记录 audit log 开发的第二阶段工作## 标签
OceanBase, MySQL, audit log, DDL, 内部表, 表结构定义

## 摘要
记录在 OceanBase 上开发 MySQL audit log 功能时，对内部表结构的分析和实现方式。重点研究了__all_ddl_operation 等系统表的表结构定义、索引初始化和代码调用关系。

## 关键概念
- __all_ddl_operation: OceanBase 内部记录 DDL 操作的系统表
- ob_inner_table_schema_def.py: 定义 OceanBase 内部表结构的 Python 配置文件
- 租户表/全局表: OceanBase 中两种不同的表空间类型，分别用于租户级和全局级数据
- GV$OB_SQL_AUDIT: OceanBase 全局 SQL 审计视图

## 关联笔记
- 01KJBZ4J03FH868Z4ZDKGPQRDY.md: 同一系列后续笔记，继续记录 audit log 开发的第二阶段工作
