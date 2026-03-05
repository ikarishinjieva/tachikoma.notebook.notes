---
note: 01KJBYXQVR8F1NJ5EW4M7DGCRA.md
title: 20230210 - gitlab->github sync failed
indexed_at: 2026-03-05T08:27:59.106850+00:00
---

## 摘要
记录 GitLab 到 GitHub 同步失败的故障信息，包含服务器地址、工作目录、锁文件位置和日志路径。用于追踪 git-syncer 服务的运行状态和问题排查。

## 关键概念
- 锁文件机制: 通过 `/tmp/.BackupLockFile` 和 `GIT_SYNC_RUNNING` 防止同步任务并发执行
- cron 日志: `/tmp/cron.log` 记录定时任务执行情况和错误信息
- git-syncer: 部署在 `/data/git-syncer` 目录的 Git 仓库同步服务

## 关联笔记
- 01KJBYDPFB6VAMXX430NJQVYGP.md: 涉及同一台服务器 (10.186.18.20) 的 VPN 网络配置和连通性排查
