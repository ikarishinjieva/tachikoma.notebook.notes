---
note: 01KJBZ9EHSQP9NTHD0TA7N2REQ.md
title: 20240131 - 对git分支中的时间和committer进行调整
indexed_at: 2026-02-27T10:55:39.093014+00:00
---

用户要求我分析一篇关于 Git 分支时间和 committer 调整的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：Git 分支操作，包括 fast-export/fast-import、修改 commit 的 author/committer 信息、时间调整、分支重命名等

2. 核心内容：
- 备份 repo
- 使用 git fast-export 导出提交
- 使用 sed 处理导出文件（修改 author/committer 信息）
- 使用 patch 应用修改
- 修改分支引用从 main 到 refine
- 使用 git fast-import 重新导入
- 强制推送到远程

3. 还包含一个单独设置环境变量进行提交的示例

4. 关联笔记分析：
- 这篇笔记主要是 Git 操作相关
- 查看其他笔记列表，大部分是 MySQL、Clickhouse、数据库相关的
- 没有明显与 Git 版本控制直接相关的笔记
- 所以关联笔记应该是「无」

现在生成元数据：

标签：git, 版本控制, fast-export, fast-import, 分支管理, commit 历史

摘要：介绍使用 git fast-export/fast-import 工具批量修改 Git 仓库中提交的时间和 committer 信息的方法。包含导出、修改、重新导入的完整流程，以及通过环境变量设置单次提交信息的技巧。

关键概念：
- git fast-export: 将 Git 仓库历史导出为文本格式
- git fast-import: 从文本格式导入数据创建 Git 历史
- committer: 实际执行提交操作的人


- author: 原始代码作者
- GIT_AUTHOR_DATE/GIT_COMMITTER_DATE: 控制提交时间的环境变量

关联笔记：无（其他笔记都是数据库相关，与 Git 操作无关）## 标签
Git, 版本控制, fast-export, fast-import, 分支管理, 提交历史

## 摘要
介绍使用 git fast-export/fast-import 工具批量修改 Git 仓库中提交的时间和 committer 信息的方法。包含导出、sed 处理、patch 应用、重新导入的完整流程，以及通过环境变量设置单次提交信息的技巧。

## 关键概念
- git fast-export: 将 Git 仓库历史导出为文本格式便于批量修改
- git fast-import: 从文本格式导入数据重建 Git 历史
- committer: 实际执行提交操作的人及时间
- author: 原始代码作者及时间
- 环境变量: GIT_AUTHOR_DATE/GIT_COMMITTER_DATE 等控制提交元数据

## 关联笔记
无
