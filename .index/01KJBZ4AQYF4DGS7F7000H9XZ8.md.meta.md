---
note: 01KJBZ4AQYF4DGS7F7000H9XZ8.md
title: 20231204 - 整理OMS的全量复制链路
indexed_at: 2026-02-27T10:40:35.059458+00:00
---

用户要求我分析这篇关于 OMS 全量复制链路的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是关于 OceanBase OMS (OceanBase Migration Service) 的全量数据复制链路的技术笔记
2. 主要描述了两个处理器：
   - 处理器1：扫描表的元数据
   - 处理器2：数据切分器，获取数据切分边界
3. 包含代码调用栈、线程信息、队列信息、Arthas 命令等
4. 涉及 Oracle 数据库的块号查询、表空间等概念

其他笔记列表主要是关于 MySQL、Clickhouse、数据库相关的技术笔记，但这篇笔记是关于 OceanBase OMS 的，与其他笔记的关联性需要判断。

让我生成索引：

标签：OMS、全量复制、数据切分、OceanBase、元数据扫描

摘要：这篇笔记整理了 OMS 全量复制链路的两个核心处理器流程。处理器 1 负责扫描源库表元数据，处理器 2 负责数据切分并生成切片边界。包含代码调用栈、线程模型、队列信息和 Arthas 调试命令。

关键概念：
- SliceTaskManager: 管理表元数据和切片任务的核心组件
- 数据切分方式：支持 All/PK/PKIncrement/Block 四种切分策略


- Block 切片：基于 Oracle 数据块（OBJECT_NUMBER/RELATIVE_FNO/BLOCK_NUMBER）进行物理切分

关联笔记：其他笔记主要涉及 MySQL、Clickhouse 等技术栈，与这篇 OceanBase OMS 的笔记没有直接关联。## 标签
OMS, 全量复制, 数据切分, OceanBase, 元数据扫描, Oracle

## 摘要
这篇笔记整理了 OMS 全量复制链路的两个核心处理器流程。处理器 1 负责扫描源库表元数据，处理器 2 负责数据切分并生成切片边界。包含代码调用栈、线程模型、队列信息和 Arthas 调试命令。

## 关键概念
- SliceTaskManager: 管理表元数据扫描和切片任务分发的核心组件
- 数据切分方式: 支持 All/PK/PKIncrement/Block 四种切分策略
- Block 切片: 基于 Oracle 数据块（OBJECT_NUMBER/RELATIVE_FNO/BLOCK_NUMBER）进行物理切分
- Bigfile Tablespace: 特殊表空间类型，需要转换块号格式

## 关联笔记
无
