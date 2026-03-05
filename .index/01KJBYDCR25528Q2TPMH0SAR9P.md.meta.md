---
note: 01KJBYDCR25528Q2TPMH0SAR9P.md
title: 20210526 - 农行 C# 程序 SSL超时
indexed_at: 2026-03-05T07:29:10.744523+00:00
---

## 摘要
C#程序连接MySQL时SSL握手超时，客户端15s超时，MySQL端10s超时。通过抓包和.Net Framework tracing分析，问题定位在Windows CTL自动更新机制导致证书验证过程卡顿，建议调整组策略关闭CTL自动刷新。

## 关键概念
- StartTLS: 在明文连接上升级为加密连接的协议扩展
- SecureChannel: .NET中处理SSL/TLS安全通信的通道类
- CTL自动更新: Windows自动更新不受信任证书列表的机制
- 证书验证: SSL握手过程中验证服务器证书有效性的步骤

## 关联笔记
无
