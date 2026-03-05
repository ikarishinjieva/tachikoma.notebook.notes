---
note: 01KJC04AN1CTFSD2RFBK4RQPFF.md
title: 20251012 - 分析isaac_ros_ess的原理 (ESS神经网络 感知深度)
indexed_at: 2026-03-05T12:08:08.599759+00:00
---

## 标签
ESS, 立体视觉，深度估计，Isaac ROS, 神经网络，Isaac Sim

## 摘要
记录 isaac_ros_ess 包的安装、配置和测试流程，包括在 ROS2 容器和 Isaac Sim 中的使用方法。分析 ESS 神经网络生成立体深度图的架构，对比传统 SGM 方法的差异。

## 关键概念
- ESS: 基于神经网络的立体深度估计模型
- 深度图：ESS 输出的核心数据，表示像素级深度信息
- TensorRT Engine: ESS 模型的加速推理引擎格式
- 点云重建：使用左相机图像 + 深度图整合生成的可视化数据

## 关联笔记
- 01KJC03F2DVR3X6WTK4RPEFBY8.md: 对比传统 SGM 方法与 ESS 神经网络的立体视觉方案差异## 标签
ESS, 立体视觉，深度估计，Isaac ROS, 神经网络，Isaac Sim

## 摘要
记录 isaac_ros_ess 包的安装、配置和测试流程，包括在 ROS2 容器和 Isaac Sim 中的使用方法。分析 ESS 神经网络生成立体深度图的架构，对比传统 SGM 方法的差异。

## 关键概念
- ESS: 基于神经网络的立体深度估计模型
- 深度图：ESS 输出的核心数据，表示像素级深度信息
- TensorRT Engine: ESS 模型的加速推理引擎格式
- 点云重建：使用左相机图像 + 深度图整合生成的可视化数据

## 关联笔记
- 01KJC03F2DVR3X6WTK4RPEFBY8.md: 对比传统 SGM 方法与 ESS 神经网络的立体视觉方案差异
