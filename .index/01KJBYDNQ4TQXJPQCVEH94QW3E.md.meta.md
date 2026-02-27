---
note: 01KJBYDNQ4TQXJPQCVEH94QW3E.md
title: 20210628 - VSS/RSS/USS/PSS 解释
indexed_at: 2026-02-27T10:06:00.403204+00:00
---

用户要求我分析一篇关于 Linux 内存测量工具 procrank 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：VSS/RSS/USS/PSS 内存指标解释，以及 procrank 工具的使用
2. 核心内容：
   - 介绍了 Linux 进程内存测量的复杂性
   - 解释了 VSS(虚拟集大小)、RSS( resident 集大小)、USS(唯一集大小)、PSS(比例集大小) 四个指标
   - 说明了 procrank 工具的来源和用途

3. 可能的标签：Linux、内存分析、procrank、VSS、RSS、USS、PSS、嵌入式

4. 关联笔记分析：
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 这个明显与内存分析相关
   - 01KJBYDP3X93594WV0FTRG0KGM.md: 20210705 - linux 的 CPU 频率 - 都是 Linux 系统层面的分析，但关联度不高
   - 其他笔记大多是关于 MySQL、Clickhouse 的，关联度较低

最相关的是「内存去哪儿了」这篇。

现在我来生成输出：

## 标签
Linux, 内存分析, procrank, VSS, RSS, PSS

## 摘要
介绍 Linux 进程内存测量的复杂性及 procrank 工具的使用方法。详细解释了 VSS、RSS、USS、PSS 四种内存指标的含义和计算方式，其中 PSS 能准确反映进程实际内存占用。
   
## 关键概念
- VSS (Virtual Set Size): 进程映射的总虚拟内存，包含未使用的内存
- RSS (Resident Set Size): 映射的物理内存，但未考虑共享页面
- USS (Unique Set Size): 进程独占的私有内存，不包含共享部分
- PSS (Proportional Set Size): 按共享进程数分摊后的内存，最准确反映实际占用

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同样涉及内存分析主题## 标签
Linux, 内存分析, procrank, VSS, RSS, PSS, 嵌入式

## 摘要
介绍 Linux 进程内存测量的复杂性及 procrank 工具的使用方法。详细解释了 VSS、RSS、USS、PSS 四种内存指标的含义和计算方式，其中 PSS 能准确反映进程实际内存占用。

## 关键概念
- VSS (Virtual Set Size): 进程映射的总虚拟内存，包含已分配但未使用的内存
- RSS (Resident Set Size): 映射的物理内存量，但未考虑进程间共享
- USS (Unique Set Size): 进程独占的私有内存， fork 时的内存代价
- PSS (Proportional Set Size): 共享内存按进程数分摊，总和等于实际内存占用

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同为内存分析主题，探讨内存使用情况
