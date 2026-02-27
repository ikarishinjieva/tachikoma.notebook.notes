---
note: 01KJBZ3M0XYJH7DR6ZWFDNNEYS.md
title: 20231105 - 判断Oracle某个SQL hang的原因
indexed_at: 2026-02-27T10:37:20.806916+00:00
---

用户要求我分析一个关于 Oracle SQL hang 原因判断的笔记，并生成索引元数据。让我分析笔记内容：

笔记内容包含：
1. `select * from dba_blockers;` - 查询 Oracle 阻塞者
2. `select serial# from v$session s where s.sid = 221` - 查询会话的 serial 号
3. `alter system kill session '221,73' immediate;` - 杀死会话

这是关于 Oracle 数据库故障排查的笔记，主要涉及会话阻塞和 Kill 会话的操作。

现在我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：从提供的列表中找 0-3 篇相关的

关于关联笔记，我看看有没有与 Oracle、数据库阻塞、会话管理相关的：
- 大部分笔记是关于 MySQL、Clickhouse 的
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 临时磁盘表相关
- 01KJBYDBNY9PYDPG9JKCJSH6CV.md: druid 连接池
- 01KJBYDD09MGYR4GA0C4KB7P9F.md: TODO 笔记

看起来没有直接关于 Oracle 阻塞的笔记，所以关联笔记应该是「无」。

让我生成输出：## 标签
Oracle, 会话阻塞, 故障排查, SQL hang, 会话管理

## 摘要
记录 Oracle 数据库中排查 SQL hang 问题的常用命令。通过查询阻塞者、获取会话信息、终止会话三步定位和解决阻塞问题。

## 关键概念
- dba_blockers: Oracle 视图，用于查询当前阻塞其他会话的会话
- v$session: Oracle 动态性能视图，存储会话信息
- kill session: Oracle 终止会话命令，需指定 sid 和 serial#

## 关联笔记
无
