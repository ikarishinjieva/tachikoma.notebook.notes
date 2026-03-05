---
note: 01KJBYHBXV2TGZE3J3CYXR3TD9.md
title: 20220213 - 浦发MGR切换问题诊断
indexed_at: 2026-03-05T07:56:17.052896+00:00
---

## 标签
MySQL, MGR, Group Replication, 问题诊断, GDB 调试, Applier

## 摘要
记录浦发银行 MGR 切换时 group_replication 插件报错导致节点离组的问题。通过 GDB 断点分析 Sql_service_interface::set_session_user 函数堆栈，定位到 Certifier 初始化阶段的 session connection 建立失败。核心问题是确认 session connection 的方向（主动端还是被动端）。

## 关键概念
- Group Replication: MySQL 高可用复制插件，支持多主或单主模式
- Applier Module: MGR 中负责回放中继日志事务的模块
- Certifier: MGR 中负责事务冲突检测和认证的组件
- mysql.session: MySQL 内部会话用户，用于插件执行特权操作

## 关联笔记
- 01KJBZ7SQMZGYSPHGCPSH9BVB6.md: 同涉及 MySQL 8.0.21 MGR 架构的锁等待和死锁问题诊断
- 01KJBZ7TPWYT2ZSTJ3GDM1APY3.md: 同涉及 MySQL 8.0.21 MGR 架构的慢 SQL 问题记录
