---
note: 01KJBYD9T2SKQ815RG0XWSD236.md
title: 20210330 - 农行 JDK导致mysql connector报错
indexed_at: 2026-03-05T07:20:03.361437+00:00
---

## 摘要
记录 MySQL Connector 8.0.20 在 IBM JDK 上因 TLS cipher suite 命名差异导致 SSL 握手失败的问题。通过设置 JVM 参数 `com.ibm.jsse2.overrideDefaultCSName=true` 使 IBM JDK 的 cipher suite 命名与 Oracle JDK 兼容，成功解决问题。

## 关键概念
- IBM JDK: IBM 实现的 Java 开发套件，其 JSSE 实现与 Oracle JDK 在 cipher suite 命名上存在差异
- Cipher Suite: TLS/SSL 协议中加密算法的组合，IBM JDK 使用 SSL_前缀而非 TLS_前缀
- SSLHandshakeException: TLS 握手失败时抛出的异常，通常由协议版本或加密套件不匹配引起
- com.ibm.jsse2.overrideDefaultCSName: IBM JDK 系统属性，用于兼容 Oracle JDK 的 cipher suite 命名规范

## 关联笔记
- 01KJBYDCR25528Q2TPMH0SAR9P.md: 同样涉及农行场景下 SSL/TLS cipher suite 兼容性问题（C# 程序）
- 01KJBZ4FDQ9H92DJ6NJAFK9EH0.md: 涉及 TLS 加密套件配置（TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA）
