---
note: 01KJC06PG4RM9T7WFDEN5HPZPN.md
title: 20251023 - 在仿真环境中, 完成一次倒水的动作
indexed_at: 2026-03-05T12:11:05.174219+00:00
---

## 摘要
阐述在 Isaac Sim 仿真环境中通过手动"导演"倒水动作来生成专家数据的必要性，为后续模仿学习奠定基础。详细介绍使用 cuMotion 进行运动规划的 6 步学习路径，从场景搭建到数据记录的全流程方法。

## 关键概念
- 模仿学习 (Imitation Learning): 通过观察专家演示学习状态 - 动作映射，比纯强化学习效率更高
- 专家数据 (Expert Data): 手动创建的高质量演示轨迹，记录关节角度、速度等状态 - 动作序列
- cuMotion: NVIDIA 的运动规划工具，用于生成平滑、最优的机械臂关节轨迹
- Sim-to-Real Gap: 仿真环境与真实世界之间的物理差异，需通过调试缩小
- 奖励稀疏问题 (Sparse Reward): 强化学习中任务复杂导致成功信号稀少，学习效率极低

## 关联笔记
- 01KJC09VMCC3G9MT6AZPAWAB54.md: 同样涉及机械臂 AI 示教和模仿学习的实践方案
- 01KJC05QF1CDNZ4BMBD9E1R7QK.md: 详细记录 Isaac Sim 与 cuMotion 的集成和使用方法
- 01KJC09E91JZFRCCYWYCZABXK2.md: 讨论机械臂运动规划，提及模仿学习作为前沿方案
