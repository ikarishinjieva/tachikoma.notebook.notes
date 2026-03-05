---
note: 01KJBZ9GRQ99REX0KKW8JGCPS9.md
title: 20240604 - 连接gitlab数据库
indexed_at: 2026-03-05T09:58:54.482913+00:00
---

## 摘要
记录通过 gitlab-psql 用户连接 GitLab 内置 PostgreSQL 数据库的命令行方法。使用 Unix socket 方式直接访问 gitlabhq_production 数据库。

## 关键概念
- gitlab-psql: GitLab 专用的 PostgreSQL 系统用户，用于管理数据库访问
- gitlabhq_production: GitLab 实例的主生产数据库名称
- Unix socket 连接: 通过本地 socket 文件而非 TCP 端口连接数据库

## 关联笔记
- 01KJBZFQYK5V76Q0KPPZG4NK8V.md: 同样记录使用 psql 命令连接 PostgreSQL 数据库的操作（Superset 后端数据库）
