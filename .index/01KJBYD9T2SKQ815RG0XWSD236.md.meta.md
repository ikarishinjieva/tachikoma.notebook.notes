---
note: 01KJBYD9T2SKQ815RG0XWSD236.md
title: 20210330 - 农行 JDK导致mysql connector报错
indexed_at: 2026-02-27T10:00:56.514128+00:00
---

用户要求我分析这篇关于农行 JDK 导致 MySQL connector 报错的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. **主题**：IBM JDK 与 MySQL connector 8.0.20 的 SSL/TLS 兼容性问题
2. **核心问题**：IBM JDK 上 MySQL connector 升级后报 SSLHandshakeException，原因是 cipher suite 命名变更
3. **解决方案**：使用 `-Dcom.ibm.jsse2.overrideDefaultCSName=true` 参数

现在我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

关联笔记分析：
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - 同一天，都是 MySQL 相关
- 01KJBYDCR25528Q2TPMH0SAR9P.md: 20210526 - 农行 C# 程序 SSL 超时 - 同是农行项目，都涉及 SSL 问题
- 01KJBYDBNY9PYDPG9JKCJSH6CV.md: 20210428 - druid 连接池 连接维护逻辑整理 - 都是数据库连接相关

让我生成简洁的元数据。## 标签
IBM JDK, MySQL Connector, SSL/TLS, Cipher Suite, 兼容性问题, 故障排查

## 摘要
记录了 MySQL Connector 从 8.0.18 升级到 8.0.20 后在 IBM JDK 上出现 SSL 握手失败的问题。通过启用 TLS 调试日志定位到 cipher suite 命名变更问题，使用 `-Dcom.ibm.jsse2.overrideDefaultCSName=true` 参数解决。

## 关键概念
- IBM JSSE2: IBM JDK 的 SSL/TLS 实现组件，cipher suite 命名规则与 Oracle JDK 不同
- Cipher Suite: TLS 握手时协商的加密算法组合，IBM JDK 将 TLS_前缀改为 SSL_
- SSLHandshakeException: SSL 握手失败异常，通常由协议或加密套件不匹配导致

## 关联笔记
- 01KJBYDCR25528Q2TPMH0SAR9P.md: 同是农行项目，都涉及 SSL 连接超时问题
- 01KJBYD9T7VWC3J9T1JMVWGM B.md: 同一天记录，都是 MySQL 相关技术笔记
- 01KJBYDBNY9PYDPG9JKCJSH6CV.md: 都是数据库连接池/连接相关的故障排查
