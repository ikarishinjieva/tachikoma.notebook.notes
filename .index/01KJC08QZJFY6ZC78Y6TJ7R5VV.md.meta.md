---
note: 01KJC08QZJFY6ZC78Y6TJ7R5VV.md
title: 20251115 - 为实际机械臂添加摄像头
indexed_at: 2026-03-05T12:14:47.407772+00:00
---

## 摘要
为实际机械臂添加摄像头组件，合并夹爪与相机的 URDF 模型，解决 STL/DAE 格式碰撞检测问题。配置 ROS2 TF 变换和 AprilTag 识别，调试相机时间轴与仿真时间同步问题，发现夹爪调度碰撞崩溃与模型格式相关。

## 关键概念
- URDF 格式转换：gripper_base 需转 STL 避免 MoveIt 碰撞检测崩溃，但 Isaac Sim 导出需保持 DAE
- TF 变换时间轴：ROS2 相机图像话题时间与 sim_time 不同步导致 TF 查询失败
- AprilTag 偏移配置：tag_to_object_offset_in_tag 与 tag_to_object_offset_in_world 两种坐标系偏移方式
- 碰撞检测冲突：DAE 格式在 MoveIt 中导致夹爪调度时偶发碰撞崩溃

## 关联笔记
- 01KJC06PG4RM9T7WFDEN5HPZPN.md: 同一机械臂项目的 URDF、Isaac Sim、MoveIt 配置与碰撞检测问题
- 01KJC08TKV281M3RRDK2CJQPCS.md: 机械臂 moveit 和 slider 环境的初始化命令配置
- 01KJC08NBRWP9P7TG4MVZS84H8.md: 机械臂笔记本基础配置与 ROS2 环境安装
