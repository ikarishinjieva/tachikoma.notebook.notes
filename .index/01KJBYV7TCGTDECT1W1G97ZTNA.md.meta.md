---
note: 01KJBYV7TCGTDECT1W1G97ZTNA.md
title: 20221117 - OMS试用
indexed_at: 2026-03-05T08:15:43.298926+00:00
---

## 标签
OMS, Docker, OceanBase, 容器部署, supervisord, 配置管理

## 摘要
记录 OMS (OceanBase Migration Service) 的安装步骤和容器结构分析。包含 Docker 镜像加载、配置文件模板、容器创建命令，以及容器内 supervisord 管理的多服务架构详解。

## 关键概念
- OMS: OceanBase 数据迁移服务，用于数据同步和迁移
- supervisord: 容器内进程管理工具，通过/etc/supervisor/conf.d/*.ini 管理多个服务
- drc_cm: 集群管理服务，基于 Jetty 启动 cm.war 提供 Web 管理界面
- config.yaml: OMS 核心配置文件，定义元数据连接、TSDB、集群节点等参数

## 关联笔记
无
