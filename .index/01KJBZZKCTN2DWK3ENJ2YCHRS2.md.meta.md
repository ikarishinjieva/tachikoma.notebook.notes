---
note: 01KJBZZKCTN2DWK3ENJ2YCHRS2.md
title: 20250908 - cursor使用海外大模型
indexed_at: 2026-02-27T11:45:58.367432+00:00
---

用户要求我分析一篇关于 Cursor 使用海外大模型的笔记，并生成索引元数据。让我仔细分析笔记内容：

笔记内容主要是：
1. 将 HTTP 协议改成 1.1
2. 配置代理设置（socks5 代理）

这是关于配置 Cursor IDE 来使用海外大模型的技术笔记。

标签应该是：Cursor, 代理配置, 大模型, IDE 配置, HTTP 协议

摘要：记录在 Cursor IDE 中配置代理以使用海外大模型的方法。包括修改 HTTP 协议版本和配置 SOCKS5 代理相关设置。

关键概念：
- HTTP 1.1: 需要使用的 HTTP 协议版本
- SOCKS5 代理: 用于访问海外服务的代理协议
- proxyStrictSSL: SSL 验证配置项
- proxySupport: 代理支持开关配置

关联笔记：浏览其他笔记列表，这篇笔记是关于 Cursor/代理配置的，其他笔记大多是关于 MySQL、Clickhouse、性能分析等，没有直接相关的。可能"公司上网"这篇有一点关联（都涉及网络配置），但关联度很低。

让我生成输出。## 标签
Cursor, 代理配置, 大模型, IDE 设置, HTTP 协议

## 摘要
记录在 Cursor IDE 中配置代理以使用海外大模型的方法。主要包括修改 HTTP 协议版本为 1.1，以及在配置文件中设置 SOCKS5 代理相关字段。

## 关键概念
- HTTP 1.1: 需要切换的 HTTP 协议版本以兼容代理服务
- SOCKS5 代理: 用于访问海外服务的本地代理协议
- proxyStrictSSL: 控制是否严格验证 SSL 证书的配置项
- proxySupport: 启用或禁用代理支持的开关配置

## 关联笔记
- 无
