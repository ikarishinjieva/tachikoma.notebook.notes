---
note: 01KJBYDNK8089C7M3W31CEM4JF.md
title: 20210628 - 公司上网
indexed_at: 2026-03-05T07:33:13.490948+00:00
---

## 摘要
记录公司网络环境下通过代理服务器访问外网的配置方法，包括 PAC 脚本和 SOCKS5 代理地址。提供 SSH 反向隧道让服务器使用本地代理，以及 pip 通过清华源安装 pysocks 的方案。

## 关键概念
- ALL_PROXY: 环境变量，用于设置命令行工具的代理地址
- SSH 反向隧道: 通过 ssh -R 将服务器端口转发到本地，使服务器能通过本地代理上网
- SOCKS5: 代理协议，支持 TCP/UDP 流量转发
- PAC 脚本: 代理自动配置脚本，根据 URL 自动选择是否走代理
- pip 镜像源: 使用国内镜像源（如清华源）加速 Python 包安装

## 关联笔记
- 01KJC08TKV281M3RRDK2CJQPCS.md: 包含类似的 SSH 连接和 http_proxy/https_proxy 环境变量配置
- 01KJC05V9SCYYZTVFCHSBJ40S3.md: 记录 UCloud 服务器的代理配置方法和 no_proxy 例外设置
- 01KJC0BM188531P5364MW0W802.md: 包含指向 10.186.16.135 的代理配置，与本笔记的 10.186.16.136 同网段
