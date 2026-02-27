---
note: 01KJBYDBNY9PYDPG9JKCJSH6CV.md
title: 20210428 - druid连接池 连接维护逻辑整理
indexed_at: 2026-02-27T10:03:14.724550+00:00
---

用户要求我分析一篇关于 Druid 连接池的笔记，并生成索引元数据。让我仔细阅读笔记内容，然后按照指定格式输出。

笔记内容分析：
1. 主要讲的是 Druid 连接池的连接维护逻辑
2. 包括创建连接、关闭连接、心跳检查、获取连接、归还连接等逻辑
3. 有一些配置建议
4. 提到了一些 Druid 的参数处理问题

标签应该包括：Druid、连接池、连接维护、心跳检查、MySQL

摘要：这篇笔记整理了 Druid 连接池的连接维护逻辑，包括创建连接、关闭连接、心跳检查等核心机制。同时提供了配置建议和已知的参数处理问题。

关键概念：
- CreatorThread: 负责创建连接的线程
- DestroyThread: 负责关闭连接的线程，定期检查连接
- 心跳检查: 通过 COM_PING 或 validationQuery 检查连接活性
- testOnBorrow/testOnReturn/testWhileIdle: 连接获取/归还/空闲时的心跳检查配置

关联笔记：从列表看，这篇笔记主要讲 Druid 连接池，其他笔记大多是关于 MySQL、Clickhouse、系统性能等。可能有关联的是：
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（都涉及 MySQL 连接相关）


- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究（涉及 MySQL 性能）

但整体来看，这篇笔记与其他笔记的关联度不高，因为其他笔记主要是关于 MySQL 内部机制、Clickhouse、系统性能等，而这篇是关于 Druid 连接池的。可能没有强关联。## 标签
Druid, 连接池, 连接维护, 心跳检查, MySQL, 配置优化

## 摘要
这篇笔记整理了 Druid 连接池的连接维护逻辑，包括 CreatorThread 创建连接、DestroyThread 关闭连接、心跳检查等核心机制。同时提供了配置建议和已知的参数处理问题。

## 关键概念
- CreatorThread: 负责创建连接的线程，根据 keepalive、minIdle、maxActive 等条件决定是否建立新连接
- DestroyThread: 定期触发连接检查的线程，负责关闭超时/空闲连接、心跳检查、泄露检测
- 心跳检查: 通过 COM_PING 或 validationQuery 检查连接活性，可在获取/归还/空闲时触发
- removeAbandoned: 连接泄露检测配置，超过超时时间且无 SQL 运行的连接会被强制关闭

## 关联笔记
- 无
