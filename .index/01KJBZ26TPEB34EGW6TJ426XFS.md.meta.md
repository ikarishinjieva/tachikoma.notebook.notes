---
note: 01KJBZ26TPEB34EGW6TJ426XFS.md
title: 20230802 - 修复gitlab损坏
indexed_at: 2026-03-05T08:56:22.816421+00:00
---

## 摘要
记录 GitLab 服务出现"get default branch: EOF"和 git clone 报错问题的排查过程。通过定位问题仓库并修复损坏的 refs/heads/master 分支引用恢复服务。

## 关键概念
- refs/heads: Git 分支引用存储位置，损坏会导致分支无法访问
- EOF 错误: 文件读取异常，表示引用文件数据损坏或为空
- git clone: Git 仓库克隆操作，报错"not our ref"表示引用验证失败

## 关联笔记
- 01KJBZ9GRQ99REX0KKW8JGCPS9.md: 同为 GitLab 运维笔记，记录连接 GitLab 数据库的方法
