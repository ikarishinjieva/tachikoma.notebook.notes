---
note: 01KJBZ4D06APPE59ND24QFT0WQ.md
title: 20231214 - 诊断xbcloud crash
indexed_at: 2026-03-05T09:23:20.493915+00:00
---

## 摘要
记录客户环境 xbcloud 从 S3 恢复备份时崩溃的诊断过程。通过编译 debug 版 openssl 定位到 ssl3_shutdown 调用 RAND_bytes 时发生段错误，与 CBC 加密算法相关。

## 关键概念
- openssl debug 编译: 启用详细符号信息以便解析崩溃堆栈
- CBC 加密算法: 导致 ssl3_shutdown 调用 RAND_bytes 的关键因素
- pthread_rwlock_wrlock: 崩溃发生的底层锁函数
- SSL_shutdown: OpenSSL 中触发崩溃的 SSL 关闭流程
- curl_easy_cleanup: libcurl 清理时触发 SSL 关闭

## 关联笔记
无
