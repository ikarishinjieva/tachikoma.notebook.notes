---
note: 01KJBZ7XD3F9GQ1H1VCB71FRMY.md
title: 20240418 - 对比Mariadb client和obclient的区别
indexed_at: 2026-03-05T09:44:14.864274+00:00
---

## 摘要
记录 MariaDB 10.4.18 与 obclient 两个客户端源码目录的 diff 对比结果。整理出仅存在于各自目录的文件及存在差异的配置文件，用于识别可合并的 commit。

## 关键概念
- MariaDB client: MySQL 分支数据库的客户端实现
- obclient: OceanBase 数据库客户端，基于 MariaDB 修改
- diff 对比: 用于识别两个源码树之间的文件和代码差异
- CMakeLists.txt: 构建配置文件，两者存在差异
- 代码合并: 目标是将 MariaDB 的有益 commit 合并进 obclient

## 关联笔记
- 01KJBZ391BKTMFGSSBDRWEWS5H.md: 涉及 MariaDB randgen 随机 SQL 生成项目，与本笔记中的 MariaDB 源码相关
- 01KJBZ449CMKM8CQ045S94KF5E.md: 使用 obclient 进行数据库压力测试，与本笔记的 obclient 客户端相关
