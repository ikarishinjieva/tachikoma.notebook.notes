---
note: 01KJBYWKGYZ1ZAP7STRSZWAVZJ.md
title: 20221207 - OMS checker代码分析
indexed_at: 2026-03-05T08:17:48.987510+00:00
---

## 摘要
记录 OMS 校验系统的启动脚本和 Java 运行环境配置。包含完整的 JVM 参数、classpath 依赖列表和 verifier 启动流程。

## 关键概念
- checker_new.sh: OMS 校验启动脚本，负责准备 Java 运行环境并调用 verifier-start
- verifier-start: 实际启动 Java 校验进程的脚本，提供 Cygwin 兼容性
- verification-core: OMS 校验核心库，包含多种数据库验证模块 (Oracle/MySQL/DB2 等)

## 关联笔记
- 01KJBZ4E6HJKFS7FGETVFA76RD.md: 同一 OMS 系统的验证链路分析，包含相似的 Java 启动参数和 verifier 配置
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: OMS 全量复制链路整理，涉及相同的 OMS 对象编号和 checkpoint 机制
- 01KJBZ45XPJ5T7FYJVGSDF5EKH.md: OMS 全量复制过程记录，包含 OMS_OBJECT_NUMBER 等相同的校验数据结构
