---
note: 01KJC09E91JZFRCCYWYCZABXK2.md
title: 20251217 - 进行机械臂+仿生手的运动规划的想法规划
indexed_at: 2026-03-05T12:19:06.602232+00:00
---

## 摘要
介绍使用 MoveIt Task Constructor (MTC) 进行机械臂+仿生手的复杂任务规划，包括统一规划、多规划器协同、灵活约束管理等核心特性。提出三阶段抓取策略（大体移动→精细靠近→抓取执行），并探讨结合 LeRobot 和视觉伺服实现由粗到精的分层控制方案。

## 关键概念
- MoveIt Task Constructor (MTC): 支持多阶段任务序列的统一规划框架，可组合不同规划器并自动管理约束继承
- 三阶段抓取策略: 将抓取任务分解为大体移动 (RRTConnect)、精细靠近 (CartesianPath)、抓取执行 (强制直线) 三个阶段
- 视觉伺服 (Visual Servoing): 基于图像的闭环控制，通过实时视觉反馈修正机械臂末端位置
- 分层控制 (Hierarchical Control): 由粗到精的控制策略，长距离导航用 MoveIt，局部精细操作用视觉伺服或 RL 策略
- 约束权重管理: 不同阶段设置不同的位置和姿态约束权重，实现约束逐步收紧

## 关联笔记
- 01KJC06PG4RM9T7WFDEN5HPZPN.md: 在仿真环境中使用 MoveIt 完成机械臂倒水动作的完整流程，涉及运动规划和轨迹执行
- 01KJC07BXFPJ5ZNJ51X8TM00TA.md: 分析六轴和七轴机械臂的关节设计差异，为机械臂选型提供参考
