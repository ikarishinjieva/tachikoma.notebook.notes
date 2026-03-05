---
note: 01KJBZR493PYXVJXASPHJTYJHE.md
title: 20250203 - 阅读论文: LayoutReader: Pre-training of Text and Layout for Reading Order Detection
indexed_at: 2026-03-05T11:20:43.812627+00:00
---

## 摘要
论文提出 LayoutReader 模型用于检测 PDF 文档的阅读顺序，核心贡献是构建了包含 50 万文档图像的 ReadingBank 基准数据集。该数据集利用 WORD 文档 XML 元数据中的阅读顺序信息与 PDF 布局自动对齐生成标注，解决了训练数据匮乏问题。

## 关键概念
- ReadingBank: 包含 50 万文档图像的大规模阅读顺序检测基准数据集
- XML 元数据阅读顺序: WORD 文档中嵌入的阅读顺序信息，与人类阅读习惯大部分类似
- 布局对齐: 将 XML 中的阅读顺序与 PDF/图像中的布局信息进行自动对齐标注

## 关联笔记
- 01KJBZHWF83XQC7AE6RQD6E3YZ.md: 同样涉及从 PDF 或图片中分离和识别内容的技术（DECIMER 化学式提取）
- 01KJBZHRTC8TFFQN4F8WP9JTJB.md: 涉及 PDF 表格解析和图像处理流程（OSRA 代码解析）
