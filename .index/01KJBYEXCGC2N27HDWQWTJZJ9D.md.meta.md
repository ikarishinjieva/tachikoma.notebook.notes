---
note: 01KJBYEXCGC2N27HDWQWTJZJ9D.md
title: 20210915 - 编译gperftools / tcmalloc
indexed_at: 2026-02-27T10:10:41.778673+00:00
---

用户要求我分析一篇关于编译 gperftools/tcmalloc 的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：20210915 - 编译 gperftools / tcmalloc
2. 内容：在 CentOS 7 虚拟机上编译 gperftools 2.9.1 的 RPM 包的步骤

需要生成：
- 标签（3-6 个）
- 摘要（2-3 句话）
- 关键概念（3-5 个）
- 关联笔记（0-3 篇，从提供的列表中选择）

分析关联笔记：
- 这篇笔记主要是关于编译 gperftools/tcmalloc（内存分配器、性能分析工具）
- 查看其他笔记，最相关的可能是一些性能相关的：
  - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（性能相关）
  - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（内存相关）
  - 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（内存相关）

但说实话，这些关联度都不是很高。gperftools/tcmalloc 是内存分配器和性能分析工具，与内存研究有一定关联。

最相关的应该是：
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（都涉及内存分析）
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（都涉及内存）

不过这些关联度确实不强，因为这篇笔记主要是编译教程，而不是内存分析本身。

现在需要生成索引元数据。
## 标签
gperftools, tcmalloc, RPM 打包, CentOS 7, 编译, 性能分析

## 摘要
记录在 CentOS 7 虚拟机上编译 gperftools 2.9.1 RPM 包的完整流程。包括依赖安装、源码准备、spec 文件修改及打包命令。

## 关键概念
- gperftools: Google 开源的性能分析工具集
- tcmalloc: 线程缓存内存分配器，gperftools 的核心组件之一
- RPM: Red Hat 系 Linux 的软件包格式
- rpm.spec: RPM 打包的配置文件，定义编译和安装规则

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 都涉及内存分析相关主题
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 都涉及内存相关的技术研究
