---
note: 01KJC029BR3Q8FBQ9WN3QRRVDE.md
title: 20251003 - 重新配置isaac和demo的笔记本
indexed_at: 2026-03-05T12:02:08.154209+00:00
---

## 标签
Isaac Sim, NVIDIA, ROS2, Docker, CUDA, 环境配置

## 摘要
记录 Isaac Sim 5.0.0  standalone 安装流程及 IOMMU 导致 CUDA 报错的解决方案（修改 GRUB 内核参数）。包含 demo 环境搭建：nvidia-container-toolkit 配置、ROS2 工作区创建、容器运行验证及 CUDA 流测试。

## 关键概念
- IOMMU: 输入输出内存管理单元，裸机 Linux 系统需禁用以避免 CUDA 图像损坏
- nvidia-container-toolkit: 使 Docker 容器能够正确使用 NVIDIA 显卡的运行时工具
- isaac_ros_common: NVIDIA Isaac ROS 通用组件仓库，提供 ROS2 开发基础框架
- cudaMallocAsync: CUDA 异步内存分配 API，需配合 CUDA 流使用

## 关联笔记
- 01KJC00P6GGFBQ7N84E68RP3C8.md: 前序笔记，记录 Isaac 环境初始搭建步骤
- 01KJC02762W97A18HXBYY70NM7.md: 参考文档，详述 Isaac Sim 中机械手教程配置流程
- 01KJC029N8APNRN6BXH70S5YQD.md: 后续笔记，记录双网络环境下 Isaac Sim 与 ROS2 demo 的配置问题
