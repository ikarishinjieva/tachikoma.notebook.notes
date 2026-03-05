---
note: 01KJC0CY3E5Z18ZPRZ0X69SH1F.md
title: 20260205 - 搭建labelme环境, 进行yolo训练
indexed_at: 2026-03-05T12:23:06.718916+00:00
---

## 摘要
记录搭建 labelme 标注环境和 YOLO 训练环境的全过程，包括 conda 环境配置、依赖安装和 SSH 桌面连接方法。提供两个 Python 脚本：labelme 转 YOLO 格式的数据转换脚本和数据集划分脚本。

## 关键概念
- labelme: 图像标注工具，用于手动标注关键点生成 JSON 格式标注文件
- YOLO-Pose: 目标检测框架的姿态估计模型，用于识别移液枪关键点
- 数据格式转换: 将 labelme 的 JSON 标注转换为 YOLO 训练所需的 TXT 格式
- 数据集划分: 按比例将标注数据划分为训练集和验证集

## 关联笔记
- 01KJC0CANQHRJ7GDJWQAST4EEW.md: 20260204 的笔记中提到"搭建本地标记环境"，直接指向本笔记
- 01KJC09CHJM2B6XARNP1VND0TR.md: 20251216 的 YOLO 尝试笔记，同属移液枪 YOLO 识别项目系列
