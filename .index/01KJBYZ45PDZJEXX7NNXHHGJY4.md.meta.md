---
note: 01KJBYZ45PDZJEXX7NNXHHGJY4.md
title: 20230503 - 如何使用图片作为LLM的上下文 (暂停)
indexed_at: 2026-03-05T08:41:32.138424+00:00
---

## 标签
OCR, 图片识别, LLM 上下文, Tesseract, Paddle OCR, 图像预处理

## 摘要
测试多种 OCR 方案（unstructured、Tesseract、Paddle OCR）将图片转换为 LLM 可理解的文本。对比了不同图片增强方法（Pillow、scikit-image）对识别效果的影响，发现二值化阈值处理是关键难点。

## 关键概念
- unstructured: 多模态文档解析库，支持图片 OCR 但识别效果有限
- Tesseract OCR: Google 开源 OCR 引擎，支持多语言但需预处理优化
- Paddle OCR: 百度开源 OCR 系统，对中文和复杂场景识别效果较好
- 图像预处理: 通过对比度增强、二值化、旋转校正提升 OCR 识别率
- Otsu 阈值: 自适应二值化算法，但对显示器反光等场景需调整

## 关联笔记
- 01KJBYYMHN0C5CA4ARZ21B6AAN.md: 测试 llama_index 处理图片作为 LLM 上下文的方案
- 01KJBZEQDEV2ES96F3T90D1B65.md: 开发 ai-tester 时通过 OCR 定位界面元素的实践
- 01KJBZJ2B4WEX7J5JS6W6BXGYH.md: Qwen2-VL 论文阅读，涉及视觉语言模型的 OCR 能力
