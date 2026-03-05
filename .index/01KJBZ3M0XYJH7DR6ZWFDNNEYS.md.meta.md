---
note: 01KJBZ3M0XYJH7DR6ZWFDNNEYS.md
title: 20231105 - 判断Oracle某个SQL hang的原因
indexed_at: 2026-03-05T09:11:56.773187+00:00
---

## 摘要
记录 Oracle 数据库 SQL hang 住时的诊断和解决方法。通过 dba_blockers 查询阻塞源，结合 v$session 定位会话，使用 kill session 终止问题会话。

## 关键概念
- dba_blockers: Oracle 视图，显示当前阻塞其他会话的阻塞者信息
- v$session: Oracle 动态性能视图，包含所有会话的详细信息
- kill session: Oracle 命令，用于强制终止指定会话
- serial#: 会话序列号，与 sid 一起唯一标识一个会话

## 关联笔记
- 01KJBZEE9JTH4RKQG664GFW88H.md: MySQL crash 分析笔记，同样涉及数据库锁等待和线程阻塞的排查方法
- 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md: 涉及 MySQL 死锁问题的知识图谱抽取，与数据库锁问题相关
