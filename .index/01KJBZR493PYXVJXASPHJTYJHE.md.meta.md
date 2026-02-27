---
note: 01KJBZR493PYXVJXASPHJTYJHE.md
title: 20250203 - 阅读论文: LayoutReader: Pre-training of Text and Layout for Reading Order Detection
indexed_at: 2026-02-27T11:30:13.448501+00:00
---

用户要求我分析一篇关于 LayoutReader 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：关于 LayoutReader 论文的阅读笔记，主要关注 PDF 阅读顺序检测
2. 核心内容：
   - 论文目标：训练模型识别 PDF 的阅读顺序
   - 关键思路：如何创建训练数据 - ReadingBank 数据集（50 万文档图像）
   - 数据来源：利用 WORD 文档的 XML 元数据中的阅读顺序信息

3. 标签应该包括：论文阅读、阅读顺序检测、LayoutReader、训练数据、PDF 布局等

4. 关键概念：
   - LayoutReader: 用于检测阅读顺序的预训练模型
   - ReadingBank: 包含 50 万文档图像的阅读顺序检测基准数据集
   - 阅读顺序检测: 识别 PDF 文档中内容的阅读先后顺序

5. 关联笔记分析：
   查看其他笔记列表，大部分是关于 MySQL、Clickhouse、系统性能等方面的笔记，与这篇论文笔记主题不太相关。
   - 01KJBYDD09MGYR4GA0C4KB7P9F.md: "TODO: Advanced Join Strategies for Large-Scale Distributed Computation" - 这也是论文相关的笔记，可以算作关联
   
   其他笔记主要是数据库、系统调试相关，与这篇论文没有直接关联。

现在生成索引元数据。
## 标签
论文阅读, 阅读顺序检测, LayoutReader, 训练数据, PDF 布局, ReadingBank

## 摘要
这篇笔记记录了 LayoutReader 论文的核心思路，该论文旨在训练一个识别 PDF 阅读顺序的模型。笔记重点关注了论文解决训练数据匮乏问题的方法——利用 WORD 文档 XML 元数据自动生成标注数据集 ReadingBank。

## 关键概念
- LayoutReader: 用于检测文档阅读顺序的文本与布局预训练模型
- ReadingBank: 包含 50 万文档图像的大规模阅读顺序检测基准数据集
- 阅读顺序检测: 识别 PDF 或图像中文档内容的正确阅读先后顺序

## 关联笔记
- 01KJBYDD09MGYR4GA0C4KB7P9F.md: 同为论文阅读笔记，涉及大规模计算相关研究
