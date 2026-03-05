---
note: 01KJC09K67G0QZ4S7KKNWK133F.md
title: 20260104 - 机械臂 moveit+ros2_control+driver的启动链路
indexed_at: 2026-03-05T12:20:51.587305+00:00
---

## 摘要
描述机械臂从启动到运行的完整链路：MoveIt 生成 robot_description（含 ros2_control 配置）→ controller_manager 加载硬件插件和控制器 → spawner 激活控制器 → move_group 通过 FollowJointTrajectory 接口下发轨迹。涵盖 5 步启动流程和运行态数据流闭环（规划→插值→read/write→状态发布）。

## 关键概念
- ros2_control: ROS2 的标准控制框架，通过硬件插件抽象物理硬件，提供统一的控制器接口
- controller_manager: ros2_control 的核心节点，负责加载/配置/激活控制器，管理硬件资源分配
- spawner: 控制器激活工具，通过调用/controller_manager 服务接口来加载和启动控制器
- 硬件接口插件: 实现 read()/write() 的插件类，连接控制器与物理硬件（如 AliciaDHardwareInterface）
- FollowJointTrajectory: MoveIt 与 JointTrajectoryController 通信的标准 Action 接口

## 关联笔记
- 01KJC06PG4RM9T7WFDEN5HPZPN.md: 详细描述 MoveIt+ros2_control 在 Isaac Sim 中的完整配置和 11 步运行机制
- 01KJC09HGP5EADCGYKZC5QCW6M.md: 包含 AliciaDHardwareInterface 和 Alicia_controller 的实际运行日志
- 01KJC08NBRWP9P7TG4MVZS84H8.md: 涉及 ros2_controllers.yaml 中刷新率配置的调整记录
