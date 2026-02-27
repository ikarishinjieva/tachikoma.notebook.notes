---
note: 01KJBZ3E7H227P22AC36KNWTM1.md
title: 20231024 - arthas 学习
indexed_at: 2026-02-27T10:36:35.435612+00:00
---

用户要求我分析一篇关于 Arthas 学习的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：Arthas（阿里巴巴开源的 Java 诊断工具）的各种功能学习
2. **主要内容**：列出了多个 Arthas 命令及其用途，包括 thread、vmtool、dump、mc、retransform、sc、trace、tt、watch 等
3. **标签**：应该提取 3-6 个关键词，如 Java 诊断、Arthas、性能分析、线程分析、字节码等
4. **摘要**：2-3 句话概括核心内容
5. **关键概念**：3-5 个核心概念及其解释
6. **关联笔记**：从提供的笔记列表中找出相关的

看关联笔记，大部分是关于 MySQL、Clickhouse、内存分析等主题。与 Arthas/Java 诊断相关的可能是：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（可能涉及 Java 内存分析）
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（涉及 JDK）
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 20210513 - 百胜 cpu 高的探索（可能涉及性能分析）

但这些关联度都不高，因为这篇笔记主要是关于 Arthas 工具的学习，而其他笔记大多是关于数据库的。

考虑到主题差异较大，我应该谨慎判断是否真的存在关联，或者标注为无关联。## 标签
Arthas, Java 诊断，性能分析，线程分析，字节码增强，调用追踪

## 摘要
本文整理了 Arthas Java 诊断工具的核心功能命令及其用途。涵盖线程分析、类搜索、方法追踪、字节码操作等诊断能力，并附有官方文档链接。

## 关键概念
- thread: 查看线程状态，定位阻塞线程和最繁忙线程
- trace: 追踪方法内部调用路径，统计各调用耗时
- retransform: 动态替换类文件实现热更新
- watch: 基于 AOP 切面观测方法参数和返回值
- tt: 记录方法调用上下文快照，支持回放和变量检查

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同属 JVM 诊断分析场景）
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错（涉及 JDK 问题排查）
