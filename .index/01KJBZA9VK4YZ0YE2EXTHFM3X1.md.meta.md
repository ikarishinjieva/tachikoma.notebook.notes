---
note: 01KJBZA9VK4YZ0YE2EXTHFM3X1.md
title: 20240621 - ChatDBA: 尝试step-back
indexed_at: 2026-03-05T10:14:40.754993+00:00
---

## 摘要
记录使用 step-back 技术将具体数据库问题抽象为通用检索问题的实验过程。包含 4 个 DBA 问题抽象案例（主从延迟、INSERT 返回值、文件打开数限制、DECIMAL 类型错误），并对比抽象前后对知识库召回效果的影响。

## 关键概念
- step-back: 将具体问题抽象为更通用形式以提升检索效果的技术
- 问题抽象: 去除具体细节、保留问题本质以获取更广泛适用的解决方案
- 主从延迟: MySQL 复制架构中从库落后主库的现象
- 大事务: 执行时间长、影响数据量大的数据库操作
- 知识库召回: 通过检索系统获取相关文档的过程

## 关联笔记
- 01KJBZAQZ6CX2KZQ4FC02BPFXS.md: 同一天记录 ChatDBA 如何使用知识类文档进行检索
- 01KJBZB3ESK010EVRSR0XJDJFA.md: ChatDBA 使用思维链和参考文档进行问题排查的尝试
- 01KJBZZYFRDE3MNYM2SP16GZ3T.md: ChatDBA 项目的代码仓库地址
