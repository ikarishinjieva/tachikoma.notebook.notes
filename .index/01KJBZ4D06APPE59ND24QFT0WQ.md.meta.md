---
note: 01KJBZ4D06APPE59ND24QFT0WQ.md
title: 20231214 - 诊断xbcloud crash
indexed_at: 2026-02-27T10:40:45.769612+00:00
---

用户希望我分析这篇关于诊断 xbcloud crash 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. **标题**: 20231214 - 诊断 xbcloud crash
2. **核心内容**: 
   - 客户在 ARM 和 x86 环境从 S3 恢复备份时，xbcloud 发生崩溃
   - 本地无法复现
   - 通过编译 debug 版本 openssl 来解析堆栈
   - 发现崩溃与 SSL 加密算法有关（CBC 类型的 DHE-RSA-AES256-SHA）
   - 涉及 pthread_rwlock_wrlock、RAND_bytes、ssl3_shutdown 等调用

3. **关键概念**:
   - xbcloud: 备份恢复工具
   - openssl: 加密库，需要 debug 版本解析堆栈
   - SSL/TLS: 加密算法影响崩溃
   - CBC 加密算法: 导致 ssl3_shutdown 调用 RAND_bytes
   - 堆栈分析: 诊断崩溃原因

4. **关联笔记**:
   - 查看提供的笔记列表，寻找与调试、崩溃、SSL、openssl 相关的
   - 01KJBYDCR25528Q2TPMH0SAR9P.md: 20210526 - 农行 C# 程序 SSL 超时 - 都涉及 SSL 问题
   - 其他笔记主要是 MySQL、Clickhouse 相关，关联度不高

现在生成元数据。## 标签
xbcloud, 崩溃诊断, openssl, SSL, 堆栈分析, CBC 加密

## 摘要
客户在 ARM 和 x86 环境从 S3 恢复备份时 xbcloud 发生崩溃，本地无法复现。通过编译 debug 版 openssl 解析堆栈，发现崩溃与 CBC 类型 SSL 加密算法（DHE-RSA-AES256-SHA）触发 ssl3_shutdown 调用 RAND_bytes 有关。

## 关键概念
- xbcloud: Percona XtraBackup 的云备份工具，用于 S3 等存储的备份恢复
- debug 版 openssl: 编译带调试信息的 openssl 以解析崩溃堆栈中的符号
- ssl3_shutdown: SSL 关闭流程，在 CBC 加密算法下会调用 RAND_bytes
- CBC 加密算法: 分组加密模式，影响 ssl3_shutdown 的行为逻辑
- pthread_rwlock_wrlock: 崩溃点，SSL 线程锁操作时发生段错误

## 关联笔记
- 01KJBYDCR25528Q2TPMH0SAR9P.md: 20210526 - 农行 C# 程序 SSL 超时（都涉及 SSL 相关问题的诊断）
