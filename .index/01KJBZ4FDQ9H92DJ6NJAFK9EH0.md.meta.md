---
note: 01KJBZ4FDQ9H92DJ6NJAFK9EH0.md
title: 20231215 - 配置minio, 将s3 API的加密套件配置为TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
indexed_at: 2026-03-05T09:23:55.660587+00:00
---

## 摘要
记录 MinIO 对象存储服务的 Docker 部署及 S3 API TLS 加密配置过程。通过 nginx 反向代理实现 TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA 加密套件，包含 OpenSSL 密钥生成、nginx SSL 配置及连接测试验证。

## 关键概念
- TLS 加密套件: 定义 TLS 连接中使用的密钥交换、加密算法和消息认证组合
- ECDHE: 椭圆曲线 Diffie-Hellman 密钥交换，提供前向安全性
- nginx SSL 代理: 使用 nginx 作为反向代理为后端服务提供 TLS 终止
- S3 API: Amazon Simple Storage Service 对象存储接口协议

## 关联笔记
- 01KJBZ4D06APPE59ND24QFT0WQ.md: 引用本笔记的 MinIO+nginx 配置方法
- 01KJBYD9T2SKQ815RG0XWSD236.md: 涉及 TLS_ECDHE_RSA 加密套件的握手分析
