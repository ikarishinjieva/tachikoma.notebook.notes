---
note: 01KJC09CHJM2B6XARNP1VND0TR.md
title: 20251216 - 尝试YOLO
indexed_at: 2026-03-05T12:18:24.152751+00:00
---

## 标签
YOLO, Roboflow, 目标检测, 移液枪识别, 图像标注, 置信度过滤

## 摘要
使用 Roboflow 平台对移液枪 top 和 neck 进行标注训练，仅 10 张图片即获得较好识别效果。通过 inference-sdk 调用 serverless 工作流，并用 OpenCV 实现本地置信度过滤与可视化。

## 关键概念
- Roboflow: 云端标注和训练平台，提供 serverless 推理工作流
- YOLO: 目标检测算法，用于识别移液枪特征点
- 置信度阈值：过滤低质量检测结果，保留高可信度预测
- inference-sdk: Roboflow 的 Python SDK，用于调用云端训练模型

## 关联笔记
- 01KJC0CANQHRJ7GDJWQAST4EEW.md: 后续使用 YOLO-seg/pose 对移液枪进行更深入的位姿分析
- 01KJC09HGP5EADCGYKZC5QCW6M.md: 提及本笔记的 YOLO 测试结果，讨论真实相机环境下的识别问题
- 01KJC09P0B1PQVA3T49QEWQ6RF.md: 梳理机械臂操作移液器的视觉方案，引用 YOLO 目标检测经验
