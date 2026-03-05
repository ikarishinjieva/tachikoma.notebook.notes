---
note: 01KJBZZKCTN2DWK3ENJ2YCHRS2.md
title: 20250908 - cursor使用海外大模型
indexed_at: 2026-03-05T11:56:43.028913+00:00
---

## 标签
Cursor, 代理配置，IDE 配置，海外模型，HTTP 协议，socks5

## 摘要
记录在 Cursor IDE 中配置代理以使用海外大模型的方法。主要包括将 HTTP 协议改为 1.1，以及在 IDE 配置文件中设置 socks5 代理相关字段（http.proxy、http.proxyStrictSSL、http.proxySupport）。

## 关键概念
- http.proxy: IDE 代理地址配置，支持 socks5 协议
- http.proxyStrictSSL: 控制是否严格验证 SSL 证书
- http.proxySupport: 启用或禁用代理支持
- socks5 代理: 本地代理协议，端口 7890
- HTTP 1.1: 需要切换的 HTTP 协议版本

## 关联笔记
- 01KJC0BM188531P5364MW0W802.md: 同样涉及 Cursor 的 proxy 配置，用于 MCP browser 功能
- 01KJC0GPN5FTPTV5X28W3WCVYW.md: 涉及 Cursor 的 marketplace 和 MCP 配置
