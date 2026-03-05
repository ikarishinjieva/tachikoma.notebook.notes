---
note: 01KJBZ83MN04S8N329FM6BMDXB.md
title: 20240423 - ChatDBA: 改善Plan的生成, 让更细节的信息放在诊断前期
indexed_at: 2026-03-05T09:46:08.398993+00:00
---

## 摘要
记录了 ChatDBA 项目中用于改善诊断 Plan 生成的原始提示词，包含多个 MySQL 连接异常故障案例的参考文档。核心案例涉及 wait_timeout 与连接池超时配置不匹配导致的"No operations allowed after connection closed"错误，以及通过 tcpdump 和 wireshark 进行网络包分析的诊断方法。

## 关键概念
- wait_timeout: MySQL 服务端空闲连接超时时间，超时后主动断开连接
- 连接池超时: 应用程序连接池的空闲连接超时设置，需与 MySQL wait_timeout 匹配
- tcpdump: Linux 网络抓包工具，用于捕获 MySQL 通信数据包
- wireshark: 网络协议分析工具，用于可视化分析抓取的 MySQL 通信包
- FIN 包: TCP 连接关闭信号，用于识别连接断开时机

## 关联笔记
- 01KJBZZYFRDE3MNYM2SP16GZ3T.md: ChatDBA 代码仓库记录，与本项目代码位置相关
- 01KJBZGDRKYMRQVC8CKDG8NVNA.md: ChatDBA 问答质量评论系统训练，同属 ChatDBA 项目体系
