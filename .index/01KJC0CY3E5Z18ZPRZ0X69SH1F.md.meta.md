---
note: 01KJC0CY3E5Z18ZPRZ0X69SH1F.md
title: 20260205 - 搭建labelme环境, 进行yolo训练
indexed_at: 2026-02-27T11:56:43.861816+00:00
---

用户要求我分析一篇关于搭建 labelme 环境和进行 YOLO 训练的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主要内容：
   - 创建 conda 环境安装 labelme
   - 配置 SSH 和桌面连接
   - 使用 labelme 进行图像标注
   - 安装 PyTorch 和 ultralytics 进行 YOLO 训练
   - 编写脚本将 labelme 标注转换为 YOLO 格式
   - 编写脚本划分训练/验证数据集

2. 标签应该是：labelme, YOLO, 目标检测，数据标注，深度学习，conda 环境

3. 摘要：记录搭建 labelme 标注环境和 YOLO 训练环境的完整流程，包括依赖安装、SSH 连接配置。提供了 labelme 转 YOLO 格式和数据集划分两个 Python 脚本。

4. 关键概念：
   - labelme: 图像标注工具
   - YOLO: 目标检测框架
   - conda: Python 环境管理工具
   - 关键点检测：标注特定点位进行姿态估计

5. 关联笔记：浏览其他笔记列表，都是关于 MySQL、Clickhouse、数据库、性能分析的内容，与这篇深度学习/目标检测的笔记没有直接关联。所以应该是「无」。

现在按照格式输出。## 标签
labelme, YOLO, 目标检测，数据标注，深度学习，环境配置

## 摘要
记录搭建 labelme 标注环境和 YOLO 训练环境的完整流程，包括 conda 环境创建、依赖安装及 SSH 桌面连接配置。提供了两个 Python 脚本：将 labelme 标注转换为 YOLO 格式，以及按训练/验证集划分数据集。

## 关键概念
- labelme: 图像标注工具，用于标注关键点和边界框
- YOLO: 实时目标检测框架，支持关键点检测任务
- conda 环境: Python 包和环境管理工具，用于隔离项目依赖
- 关键点检测: 标注图像中特定点位（如 top/bottom/hook）用于姿态估计

## 关联笔记
无
