---
note: 01KJBZ4FXPMAFNKRB7WKZED2CX.md
title: 20231217 - 进行oceanbase编译
indexed_at: 2026-03-05T09:24:22.232109+00:00
---

## 摘要
记录在 Docker 容器中编译和启动 OceanBase 数据库的完整流程，包括代码克隆、镜像拉取、容器启动、编译命令、目录初始化、observer 启动参数及数据库初始化 SQL。

## 关键概念
- ob-dev: OceanBase 开发用 Docker 镜像，包含编译和运行环境
- build.sh: OceanBase 源码编译脚本，支持 init、debug 等模式
- observer: OceanBase 数据库核心服务进程
- bootstrap: OceanBase 集群初始化命令，用于注册 zone 和 server

## 关联笔记
- 01KJBZ9MSBPXE97PBVX7FSVN5P.md: 关于打包 OceanBase 代码时排除__MACOSX 文件夹的 zip 命令
- 01KJBZ7XD3F9GQ1H1VCB71FRMY.md: 关于 obclient 与 MariaDB 客户端源码差异对比
