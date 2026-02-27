---
note: 01KJBYWKGYZ1ZAP7STRSZWAVZJ.md
title: 20221207 - OMS checker代码分析
indexed_at: 2026-02-27T10:21:00.950038+00:00
---

用户需要我分析一篇关于 OMS checker 代码分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 OMS (可能是 OceanBase Migration Service) 校验工具的分析笔记
2. 记录了启动脚本、Java 运行环境、进程启动参数、classpath 等详细信息
3. 涉及多种数据库支持（MySQL、Oracle、OceanBase、DB2 等）

标签应该包括：OMS、校验工具、Java、数据库迁移等

摘要需要概括核心内容

关键概念需要从笔记中提取

关联笔记需要判断是否有相关的笔记，从列表中看，大部分是 MySQL、Clickhouse、数据库相关的笔记，但这篇主要是关于 OMS 校验工具的，可能关联度不高。让我看看有没有相关的：
- 20210404 - 农行 JDK 导致 mysql connector 报错 - 这个涉及 JDK 和 MySQL connector，有一定关联
- 其他大部分是 MySQL 内部机制、Clickhouse 等，关联度较低

我来生成索引元数据。
## 标签
OMS, 校验工具, Java 进程, 数据库迁移, 代码分析

## 摘要
记录了 OMS checker 校验工具的启动流程和脚本调用链路。详细列出了 Java 进程的启动参数、GC 配置及完整的 classpath 依赖清单。

## 关键概念
- checker_new.sh: OMS 校验启动脚本，负责准备 Java 运行环境
- verifier-start: 被调用的实际启动脚本，支持 cygwin 兼容
- verification-core: 校验核心 jar 包，支持多种数据库类型
- JVM 参数: 配置堆内存、GC 日志、错误文件等运行参数

## 关联笔记
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同涉及 JDK 与 MySQL connector 相关问题
