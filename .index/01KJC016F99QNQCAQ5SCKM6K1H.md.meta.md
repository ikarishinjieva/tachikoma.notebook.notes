---
note: 01KJC016F99QNQCAQ5SCKM6K1H.md
title: 20250928 - isaac 机器人定义
indexed_at: 2026-03-05T12:00:56.416377+00:00
---

## 摘要
讲解 Isaac Sim 中机器人定义的层次结构，从物理实体（连杆、碰撞体、关节）到可控机器（驱动器、物理属性、关节根），形成完整的仿真机器人模型。定义了机器人=连杆 + 碰撞体 + 关节 + 驱动器 + 物理属性的完整集合。

## 关键概念
- 连杆 (Links): 机器人的刚性组成部分，构成骨骼结构
- 驱动器 (Drives): 施加在关节上的力或运动指令来源，分为位置/速度/力矩驱动
- 关节根 (Articulation Root): 告诉仿真器所有连杆关节构成一个机器人，启用高效动力学计算
- 位置驱动 (Position Drive): 控制关节达到具体目标位置或角度
- 速度驱动 (Velocity Drive): 控制关节以持续的目标速度运动

## 关联笔记
- 01KJC06PG4RM9T7WFDEN5HPZPN.md: 同属 Isaac Sim 机器人操作流程，涉及关节驱动和 Articulation 配置
- 01KJC00BEM0DA05SE31P38XF7B.md: 具身智能仿真平台概述，涵盖物理引擎和机器人模型技术栈
- 01KJC03VRF6SMQ2J69DXQ5SBTA.md: Isaac ROS nvblox 路径规划，涉及机器人运动控制
