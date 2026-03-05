---
note: 01KJBZ4J03FH868Z4ZDKGPQRDY.md
title: 20231226 - 在Oceanbase上, 开发MySQL的audit log [2]
indexed_at: 2026-03-05T09:25:01.332729+00:00
---

## 摘要
记录在 OceanBase 上开发 MySQL 审计日志模块的需求分析与模块设计，包括国测合规要求和代码量指标。规划了从内部表实验到完整审计功能的 6 步开发路线。

## 关键概念
- Audit Manager: 审计管理器，负责管理控制整个审计过程、配置审计规则
- Audit Module: 审计模块，负责监听数据库操作事件并执行审计
- Audit Logging Component: 审计日志组件，负责将审计事件存储到数据库表
- 审计事件: 包括 SQL 语句、用户名、IP 地址、时间戳等审计信息

## 关联笔记
- 01KJBZ4GNWYCPGNN5HXQ8PS58N.md: 同一系列前篇，记录 OceanBase 内部表调研和审计相关表结构分析
- 01KJBZ4FXPMAFNKRB7WKZED2CX.md: OceanBase 编译环境搭建，包含 actiondb-audit 项目克隆和容器配置
