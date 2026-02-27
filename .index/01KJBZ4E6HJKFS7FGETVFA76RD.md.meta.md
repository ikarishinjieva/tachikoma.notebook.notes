---
note: 01KJBZ4E6HJKFS7FGETVFA76RD.md
title: 20231205 - 整理OMS的全量验证链路
indexed_at: 2026-02-27T10:40:52.201729+00:00
---

用户要求我分析一篇关于 OMS 全量验证链路的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 OceanBase 迁移服务 (OMS) 全量数据验证的技术笔记
2. 包含工作目录、进程信息、配置文件
3. 有概念解释：SourceGroup、InfoExchanger、CacheCollection 等
4. 有结构梳理：Split 阶段的流程

需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从提供的列表中选择）

关于关联笔记，我需要看是否有相关的主题：
- 这篇笔记主要是关于 OMS 数据验证、数据迁移的
- 查看其他笔记标题，似乎没有直接相关的
- 可能相关的：华夏银行迁移工具评估（01KJBYDACWHS9VPYASX7JF6Q61.md）- 都是关于数据迁移工具的

让我生成输出。
## 标签
OMS, 数据验证, 全量同步, Oracle, OceanBase, 数据迁移

## 摘要
记录 OMS 全量验证链路的配置参数、进程信息和核心架构。包含 Split 阶段的 SourceGroup、切片划分器、数据交换器等组件的工作流程。

## 关键概念
- SourceGroup: 包含源端和目标端多个数据源的组合
- TriggerSliceProvider/PassiveSliceProvider: 主动/被动切片划分器，负责将源端数据分片
- InfoExchanger/SliceExchanger: 汇率中心，处理源端和目标端之间的差异和数据交换
- CacheCollection: 用于 Split 和 Verify 过程中的数据存储
- ConditionSourceConnectTask: 针对单个数据源获取数据的线程

## 关联笔记
- 01KJBYDACWHS9VPYASX7JF6Q61.md: 同为数据迁移工具评估相关笔记
