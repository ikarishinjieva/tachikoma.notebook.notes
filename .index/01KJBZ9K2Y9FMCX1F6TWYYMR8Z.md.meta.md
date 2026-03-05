---
note: 01KJBZ9K2Y9FMCX1F6TWYYMR8Z.md
title: 20240605 - git: cherry-pick 保持时间戳的脚本
indexed_at: 2026-03-05T09:59:30.418993+00:00
---

## 标签
git, cherry-pick, 时间戳, 脚本, 提交历史, 版本控制

## 摘要
一个 Bash 脚本用于在 cherry-pick 提交时保留原始作者的提交者信息和时间戳。通过设置 GIT_AUTHOR_DATE 和 GIT_COMMITTER_DATE 环境变量，确保迁移后的提交保持原始时间线。

## 关键概念
- GIT_AUTHOR_DATE: 设置作者日期的环境变量
- GIT_COMMITTER_DATE: 设置提交者日期的环境变量
- cherry-pick --no-commit: 选择提交但不立即提交，允许手动控制提交参数
- --reuse-message: 复用原始提交信息

## 关联笔记
- 01KJBZ9EHSQP9NTHD0TA7N2REQ.md: 同样涉及使用 GIT_AUTHOR_DATE 和 GIT_COMMITTER_DATE 环境变量调整 git 提交时间和 committer
