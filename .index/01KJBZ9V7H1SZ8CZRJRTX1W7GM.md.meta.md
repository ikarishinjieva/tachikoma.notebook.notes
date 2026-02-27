---
note: 01KJBZ9V7H1SZ8CZRJRTX1W7GM.md
title: 20240613 - 服务器的docker pull访问本地的翻墙代理
indexed_at: 2026-02-27T10:58:22.040469+00:00
---

用户要求我分析一篇关于服务器 Docker 代理配置的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：关于如何在服务器上配置 Docker 通过本地代理进行翻墙（访问外网）
2. 核心步骤：
   - 配置本地 clash 代理接受局域网连接
   - SSH 反向隧道将本地代理端口映射到服务器
   - 修改 Docker systemd 配置添加代理环境变量
   - 刷新配置使 docker pull 可以翻墙

3. 标签应该包括：Docker、代理、SSH 隧道、翻墙、配置等

4. 关联笔记分析：
   - 01KJBYDNK8089C7M3W31CEM4JF.md: 20210628 - 公司上网 - 这个可能有关联，都是关于网络/上网配置
   - 其他笔记主要是关于 MySQL、Clickhouse、性能分析等，与这篇笔记关联不大

让我生成符合格式的元数据。## 标签
Docker, 代理配置, SSH 隧道, 网络翻墙, systemd

## 摘要
介绍如何通过 SSH 反向隧道将本地 Clash 代理映射到远程服务器，使服务器 Docker 能够翻墙拉取镜像。主要步骤包括配置本地代理接受局域网连接、建立 SSH 反向隧道、修改 Docker systemd 服务添加代理环境变量。

## 关键概念
- SSH 反向隧道: 通过 SSH -R 参数将本地端口映射到远程服务器
- Docker 代理: 通过环境变量 HTTP_PROXY/HTTPS_PROXY 配置 Docker 的网络代理
- systemd 服务配置: 修改/lib/systemd/system/docker.service 添加环境变量
- Clash 局域网代理: 开启接受局域网连接允许其他设备使用本地代理

## 关联笔记
- 01KJBYDNK8089C7M3W31CEM4JF.md: 20210628 - 公司上网（同为网络代理/翻墙相关配置）
