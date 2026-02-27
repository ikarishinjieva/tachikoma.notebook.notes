---
note: 01KJBZ9K2Y9FMCX1F6TWYYMR8Z.md
title: 20240605 - git: cherry-pick 保持时间戳的脚本
indexed_at: 2026-02-27T10:56:23.638280+00:00
---

用户要求我分析一个关于 git cherry-pick 脚本的笔记，并生成索引元数据。让我分析内容：

1. 笔记内容是一个 bash 脚本，用于在 git cherry-pick 时保持原始提交的时间戳和作者信息
2. 脚本功能：
   - 获取原始提交的作者信息和日期
   - 设置环境变量 GIT_COMMITTER_DATE 和 GIT_AUTHOR_DATE
   - 执行 cherry-pick --no-commit
   - 使用 --reuse-message 保留提交信息

3. 标签应该是：git, cherry-pick, 时间戳, 脚本, bash 等

4. 摘要：这是一个用于在 git cherry-pick 操作时保留原始提交时间戳和作者信息的 bash 脚本

5. 关键概念：git cherry-pick, GIT_AUTHOR_DATE, GIT_COMMITTER_DATE 等

6. 关联笔记：查看其他笔记列表，主要是关于 MySQL、Clickhouse、数据库相关的，没有与 git 相关的笔记，所以应该是「无」

让我按照格式输出。
## 标签
git, cherry-pick, 时间戳, bash 脚本, 版本控制

## 摘要
这是一个用于在 git cherry-pick 操作时保留原始提交时间戳和作者信息的 bash 脚本。脚本通过设置 GIT_AUTHOR_DATE 和 GIT_COMMITTER_DATE 环境变量，确保 cherry-pick 后的提交保持原始的提交者和时间信息。

## 关键概念
- GIT_AUTHOR_DATE: 设置提交作者日期的环境变量
- GIT_COMMITTER_DATE: 设置提交者日期的环境变量
- git cherry-pick --no-commit: 将变更应用到工作区但不自动提交
- --reuse-message: 复用原始提交信息的选项

## 关联笔记
无
