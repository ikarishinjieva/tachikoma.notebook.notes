---
note: 01KJBZ9EHSQP9NTHD0TA7N2REQ.md
title: 20240131 - 对git分支中的时间和committer进行调整
indexed_at: 2026-03-05T09:56:59.201387+00:00
---

## 摘要
记录使用 git fast-export/fast-import 工具批量修改 Git 仓库中 commit 的 author/committer 信息和时间戳的操作流程。包含通过 sed 和 patch 修改导出文件，以及使用环境变量指定提交信息的单条 commit 修改方法。

## 关键概念
- git fast-export: 将 Git 仓库的提交历史导出为流格式文件
- git fast-import: 从流格式文件导入并提交到 Git 仓库
- GIT_AUTHOR_DATE: 设置 commit 作者日期的环境变量
- GIT_COMMITTER_DATE: 设置 commit 提交者日期的环境变量

## 关联笔记
- 01KJBZ26TPEB34EGW6TJ426XFS.md: 涉及 Git 仓库分支损坏修复问题，与 Git 分支操作相关
