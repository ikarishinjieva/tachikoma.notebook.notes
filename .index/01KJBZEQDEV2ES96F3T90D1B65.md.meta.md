---
note: 01KJBZEQDEV2ES96F3T90D1B65.md
title: 20240824 - 开发ai-tester
indexed_at: 2026-03-05T10:37:24.215533+00:00
---

## 摘要
记录开发 ai-tester 过程中遇到的 4 个核心问题：元素定位、全屏截图、屏幕内容描述、下一步动作决策。分析了 Playwright 的局限性及 Gemini 模型在网页结构理解上的偏差，提出 ai-tester 需要解决的三大优势方向。

## 关键概念
- 元素定位: 通过图片可见特征（如方位词、像素位置）识别网页元素
- 全屏截图: 解决 Playwright 仅截取可视区的问题，需触发 DOM 计算才能获取完整页面
- 屏幕描述: 用多模态模型解析截图描述网页结构和可操作元素
- 动作决策: 基于页面分析推荐下一步操作（Click/Fill）并标注概率

## 关联笔记
- 01KJBZEMFNMX1FSDWMKCD4DB1A.md: 前置笔记，记录 Playwright 测试的基础代码和环境配置
- 01KJBZF2KA4QBXY0P2JV6DV780.md: 后续笔记，继续开发 ai-tester 的迭代记录
