---
note: 01KJC09552D537B0DQ1NXPDZRF.md
title: 20251204 - 使用uhand操作移液枪
indexed_at: 2026-03-05T12:17:31.924400+00:00
---

## 摘要
记录使用 Linker Hand O6 灵巧手操作移液枪的方案调研与仿真集成过程。解决了 URDF 合并、惯量配置缺失、刚体嵌套等仿真问题，完成机械臂与灵巧手在 Isaac Sim 中的联合仿真配置。

## 关键概念
- Linker Hand O6: 6 主动自由度的轻量级灵巧手，重量 300g+
- URDF Xacro: 用于模块化机器人描述文件，支持组件include和joint连接
- 刚体嵌套问题: 多个RigidBodyAPI在层级中嵌套导致仿真异常
- 堵转检测: 通过电流/力矩反馈判断关节接触物体的方法

## 关联笔记
- 01KJC0DB69M3KWBB40J77757B3.md: 移液枪抓握探索的实际测试记录和思维链分析
- 01KJC09VMCC3G9MT6AZPAWAB54.md: Cursor控制灵巧手的代码实现和认知流程
- 01KJC08NBRWP9P7TG4MVZS84H8.md: mycobot机械臂与Isaac Sim的集成配置
