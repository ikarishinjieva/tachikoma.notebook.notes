---
note: 01KJC03F2DVR3X6WTK4RPEFBY8.md
title: 20251010 - 分析isaac_ros_stereo_image_proc的原理 (SGM生成点云)
indexed_at: 2026-03-05T12:06:50.357813+00:00
---

## 标签
Isaac ROS, 立体视觉, 点云生成, ROS2, TF 坐标系, SGM

## 摘要
记录 isaac_ros_stereo_image_proc 在 Isaac Sim 环境中的安装、配置和运行流程，包含完整的 launch 命令和环境变量设置。详细分析了点云无法显示的问题根源（目标坐标系 chassis_link 不存在）及解决方案（在 RViz 中切换到 base_link 坐标系）。

## 关键概念
- isaac_ros_stereo_image_proc: NVIDIA Isaac ROS 立体视觉处理包，用于从双目图像生成视差图和点云
- TF 坐标系变换: ROS2 中用于描述不同坐标系之间位置和姿态关系的机制
- SGM (Semi-Global Matching): 半全局匹配算法，用于计算双目图像的视差图
- RViz: ROS 可视化工具，用于显示点云、坐标系等机器人数据
- frame_id: ROS 消息中标识数据所属坐标系的字段

## 关联笔记
- 01KJC09C6KBEC83Z6P9EX5Y4F9.md: 双目摄像头调试笔记，包含 stereo_image_proc 的 disparity_node 和 point_cloud_node 手动运行命令
- 01KJC08TKV281M3RRDK2CJQPCS.md: Isaac ROS 容器环境初始化命令，包含相同的 ROS_DOMAIN_ID 和 CycloneDDS 配置
