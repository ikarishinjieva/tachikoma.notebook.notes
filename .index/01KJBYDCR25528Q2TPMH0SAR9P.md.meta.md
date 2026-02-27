---
note: 01KJBYDCR25528Q2TPMH0SAR9P.md
title: 20210526 - 农行 C# 程序 SSL超时
indexed_at: 2026-02-27T10:04:08.432760+00:00
---

用户要求我分析一篇关于农行 C# 程序 SSL 超时问题的笔记，并生成索引元数据。让我仔细阅读笔记内容，提取关键信息。

笔记核心内容：
1. 问题：C# 程序在连接 MySQL 时，在 StartTLS 过程中 hang 住，导致超时
2. 诊断过程：使用 .NET Framework 4.7 的 tracing 功能分析 SSL 握手过程
3. 根因分析：怀疑是 Windows 的 CTL（Certificate Trust List）自动更新机制导致证书验证过程卡顿
4. 解决方案：建议调整组策略关闭 CTL 自动刷新机制

标签应该包括：C#、SSL、MySQL、证书验证、超时、Windows

摘要：描述问题和解决方案

关键概念：
- StartTLS: MySQL 连接时的 SSL 握手协议
- CTL (Certificate Trust List): Windows 证书信任列表自动更新机制
- .NET Framework tracing: 用于调试 SSL 连接问题的日志功能
- SSL 证书验证: 客户端验证服务器证书的过程

关联笔记：
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（同样是农行 MySQL 连接问题，但这次是 JDK 相关）

让我生成输出。## 标签
C#, SSL, MySQL, 证书验证, 超时, Windows

## 摘要
C# 程序连接 MySQL 时在 StartTLS 阶段 hang 住导致超时，通过 .NET Framework tracing 和抓包分析定位问题。根因怀疑是 Windows CTL 自动更新机制导致 SSL 证书验证过程卡顿，建议通过组策略关闭该机制。

## 关键概念
- StartTLS: MySQL 连接时用于升级到 SSL/TLS 加密的协议握手过程
- CTL (Certificate Trust List): Windows 自动更新不受信任证书列表的机制，可能导致验证延迟
- .NET Framework tracing: 用于调试 SSL/TLS 连接问题的日志跟踪功能
- SSL 证书验证: 客户端验证服务器证书有效性及信任链的过程

## 关联笔记
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同为农行 MySQL 连接问题，但该笔记是 JDK 导致 connector 报错
