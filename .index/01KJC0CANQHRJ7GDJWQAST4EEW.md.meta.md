---
note: 01KJC0CANQHRJ7GDJWQAST4EEW.md
title: 20260204 - 对机械臂/手 进行 AI示教AI 的尝试二 - yolo seg/pose
indexed_at: 2026-03-05T12:22:49.753111+00:00
---

## 标签
YOLO, 图像分割, 姿态估计, 移液枪识别, Roboflow, 机械臂示教

## 摘要
测试 YOLO-seg 成功实现移液枪分割和位姿估算，泛化性良好；YOLO-pose 因数据量小、图片增强干扰导致关键点识别精度低（mAP50 仅 0.185），需更大数据集。结论：采用 YOLO-seg+ 几何算法确定位姿更可行。

## 关键概念
- YOLO-seg: 实例分割模型，小数据集即可实现良好分割效果
- YOLO-pose: 姿态估计模型，需大量标注数据才能达到可用精度
- mAP50: 标记与目标 50% 重合的及格率指标
- Roboflow: 在线标注和模型训练平台，提供图片增强功能
- 指勾识别: 移液枪挂钩关键点的视觉定位问题

## 关联笔记
- 01KJC0CY3E5Z18ZPRZ0X69SH1F.md: 下一篇，搭建本地 labelme 环境进行 YOLO 训练
- 01KJC0DB69M3KWBB40J77757B3.md: 后续尝试用 Heatmap 类算法替代 YOLO-pose 提高关键点识别精度
- 01KJC09VMCC3G9MT6AZPAWAB54.md: 提及使用 YOLO-seg/pose 估算 6D 位姿的方法参考
