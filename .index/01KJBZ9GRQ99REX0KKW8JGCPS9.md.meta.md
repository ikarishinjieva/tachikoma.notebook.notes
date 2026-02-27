---
note: 01KJBZ9GRQ99REX0KKW8JGCPS9.md
title: 20240604 - 连接gitlab数据库
indexed_at: 2026-02-27T10:56:10.766235+00:00
---

用户要求我分析一篇关于连接 GitLab 数据库的笔记，并生成索引元数据。让我分析内容：

笔记内容是一个命令，用于连接 GitLab 的 PostgreSQL 数据库。

标签应该包括：GitLab, PostgreSQL, 数据库连接，运维相关

摘要：描述这个命令的用途

关键概念：GitLab, PostgreSQL, 数据库连接等

关联笔记：查看其他笔记列表，看看有没有相关的。其他笔记主要是关于 MySQL、Clickhouse、性能分析等，没有直接关于 GitLab 或 PostgreSQL 的，所以关联笔记应该是「无」

让我生成输出：## 标签
GitLab, PostgreSQL, 数据库连接, 运维命令, 生产环境

## 摘要
记录连接 GitLab 内置 PostgreSQL 数据库的命令。通过 gitlab-psql 用户访问 gitlabhq_production 生产数据库。

## 关键概念
- gitlab-psql: GitLab 专用的 PostgreSQL 系统用户
- gitlabhq_production: GitLab 的生产数据库名称
- Unix Socket 连接: 通过本地 socket 文件连接数据库

## 关联笔记
无
