---
note: 01KJBZ45XPJ5T7FYJVGSDF5EKH.md
title: 20231128 - 整理OMS的全量复制链路 的过程
indexed_at: 2026-02-27T10:39:58.757762+00:00
---

用户要求我分析这篇关于 OMS 全量复制链路的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题是关于整理 OMS（Oceanbase Migration Service）的全量复制链路
2. 包含进程信息（connector 命令行和主目录）
3. 火焰图分析（使用 async-profiler）
4. 主启动器配置和 checkpoint 信息

关键内容：
- OMS connector 组件
- 全量数据复制/传输
- 火焰图性能分析
- checkpoint 机制
- sourceTask/sinkTask 线程
- Oracle 到 OceanBase 的数据迁移

标签应该包括：OMS、全量复制、数据迁移、性能分析、connector 等

关联笔记方面，我看了一下其他笔记列表，主要是关于 MySQL、Clickhouse、性能分析等主题。这篇笔记关于 OMS 数据迁移，与列表中的笔记关联度不高。可能稍微相关的是：
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（都涉及 connector）
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（都涉及性能分析）

但关联度都不高，所以我应该写「无」或者选择最相关的 1-2 篇。

现在我来生成输出。## 标签
OMS, 全量复制, 数据迁移, 性能分析, connector, checkpoint

## 摘要
记录 OMS 全量复制链路的进程配置与火焰图分析过程。分析了 connector 组件的启动参数、主目录结构及 checkpoint 状态，识别出 sourceTask 和 sinkTask 两类活跃线程。

## 关键概念
- connector: OMS 数据复制的核心组件，负责源端和目标端的数据传输
- checkpoint: 记录全量复制进度的机制，保存表级别的分片完成状态
- 火焰图: 使用 async-profiler 生成的性能分析图，用于识别热点线程
- splitType BLOCK: 数据分片类型，按数据块维度进行切分

## 关联笔记
无
