---
note: 01KJBZ9V7H1SZ8CZRJRTX1W7GM.md
title: 20240613 - 服务器的docker pull访问本地的翻墙代理
indexed_at: 2026-03-05T10:04:54.559668+00:00
---

## 摘要
记录通过 SSH 反向隧道将本地 Clash 代理端口映射到服务器，使服务器 docker pull 能访问外网。核心步骤包括开启 Clash 局域网访问、建立 SSH 反向隧道、修改 Docker systemd 配置添加 HTTP/HTTPS 代理环境变量。

## 关键概念
- SSH 反向隧道: 通过 `ssh -R` 将本地端口映射到远程服务器，实现服务器访问本地网络服务
- Docker 代理配置: 在 systemd service 文件中添加 Environment 变量设置 HTTP/HTTPS 代理
- Clash 局域网访问: 开启代理软件接受局域网连接功能，允许外部设备通过本机代理上网

## 关联笔记
- 01KJBZJ3GVHGXNXSFNN2B05Q2V.md: 同样使用 SSH 反向隧道转发 Clash 代理端口到服务器
- 01KJBZX25RKXAHW2N026JFEN5X.md: 记录服务器端安装 clash-for-linux 的配置方法
- 01KJBYDNK8089C7M3W31CEM4JF.md: 类似的 SSH 反向代理配置，使用 7890 端口转发
