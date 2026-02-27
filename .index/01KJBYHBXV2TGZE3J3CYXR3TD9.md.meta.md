---
note: 01KJBYHBXV2TGZE3J3CYXR3TD9.md
title: 20220213 - 浦发MGR切换问题诊断
indexed_at: 2026-02-27T10:15:53.288331+00:00
---

用户要求我分析一篇关于 MySQL Group Replication 问题诊断的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **标题**：20220213 - 浦发 MGR 切换问题诊断
2. **核心内容**：
   - MySQL Group Replication 报错日志
   - 使用 GDB 调试定位到 Sql_service_interface::set_session_user 函数
   - 核心问题是 session connection 的方向问题（主动端还是被动端）

3. **标签候选**：MySQL, Group Replication, MGR, GDB 调试，故障诊断，session connection

4. **关联笔记分析**：
   - 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: gdb 只停止触发断点的线程 - 与 GDB 调试相关
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 与 MySQL 诊断相关
   - 其他笔记大多是 Clickhouse、内存、刷盘等主题，关联度不高

让我生成符合格式的元数据。
## 标签
MySQL, Group Replication, MGR, GDB 调试，故障诊断，session connection

## 摘要
记录浦发银行 MySQL Group Replication 切换时的报错问题分析过程。通过 GDB 断点定位到 Sql_service_interface::set_session_user 函数，核心疑问是 session connection 的方向（主动端还是被动端）。

## 关键概念
- Group Replication: MySQL 原生高可用复制插件，支持多主和单主模式
- Sql_service_interface::set_session_user: GR 插件内部建立会话连接的函数
- Certifier: GR 中负责事务认证的组件，初始化时需要建立内部会话
- Applier_module: GR 中负责应用事务的模块

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: gdb 只停止触发断点的线程（同为 GDB 调试 MySQL 相关）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用（同为 MySQL 诊断工具）
