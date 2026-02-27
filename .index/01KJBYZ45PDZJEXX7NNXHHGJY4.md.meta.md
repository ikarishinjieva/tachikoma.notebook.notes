---
note: 01KJBYZ45PDZJEXX7NNXHHGJY4.md
title: 20230503 - 如何使用图片作为LLM的上下文 (暂停)
indexed_at: 2026-02-27T10:28:35.930813+00:00
---

用户希望我分析这篇关于图片 OCR 识别的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20230503 - 如何使用图片作为 LLM 的上下文 (暂停)
2. 主要内容：测试了多种 OCR 方案来识别图片中的文字
   - unstructured 方案
   - Tesseract OCR
   - Tesseract OCR + Pillow 图片增强
   - Paddle OCR
   - Tesseract OCR + scikit-image 图片增强

3. 核心内容：对比各种 OCR 工具对数据库截图的识别效果，发现二值化阈值问题

4. 标签应该是：OCR、图片识别、Tesseract、PaddleOCR、图像处理、LLM 上下文

5. 摘要：测试了多种 OCR 方案（unstructured、Tesseract、PaddleOCR）来识别图片中的文字内容，用于作为 LLM 的上下文。发现各方案对数据库截图的识别效果有限，二值化阈值需要调整。

6. 关键概念：OCR、Tesseract、PaddleOCR、图像增强、二值化

7. 关联笔记：从列表中看，这篇笔记主要是 OCR 和图片处理相关，与列表中的 MySQL、Clickhouse 等笔记关联度不高。可能没有直接关联的笔记。

现在我来生成输出。## 标签
OCR, 图片识别, Tesseract, PaddleOCR, 图像增强, LLM 上下文

## 摘要
测试了多种 OCR 方案（unstructured、Tesseract、PaddleOCR）来识别图片中的文字，用于作为 LLM 的上下文输入。各方案对数据库截图的识别效果均不理想，主要问题在于二值化阈值将显示器反光误识别为留白。

## 关键概念
- OCR: 光学字符识别，将图片中的文字转换为可编辑文本
- Tesseract: Google 开源的 OCR 引擎，支持多种语言
- PaddleOCR: 百度开源的 OCR 工具库，对中文识别效果较好
- 图像增强: 通过对比度、亮度调整等预处理提升 OCR 识别率
- 二值化: 将灰度图像转换为黑白图像，阈值选择影响识别效果

## 关联笔记
无
