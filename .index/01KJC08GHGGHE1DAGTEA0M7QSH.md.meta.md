---
note: 01KJC08GHGGHE1DAGTEA0M7QSH.md
title: 20251105 - 在Isaac Sim中, 获取选择Mesh的世界系坐标
indexed_at: 2026-03-05T12:13:01.716823+00:00
---

## 摘要
提供 Isaac Sim 中获取选中 Mesh Prim 世界坐标的完整 Python 脚本，包含矩阵分解、坐标变换和世界尺寸计算。通过 USD API 读取 Mesh 点云数据，结合 XformCache 计算局部到世界的变换矩阵，输出位置、四元数旋转和 SVD 分解的缩放参数。

## 关键概念
- UsdGeom.XformCache: USD 提供的变换缓存工具，用于高效计算 Prim 的局部到世界坐标变换
- Gf.Matrix4d: Pixar USD 的 4x4 齐次变换矩阵类型，支持行优先/列优先转换
- SVD 分解: 通过奇异值分解从变换矩阵中分离旋转和缩放分量
- metersPerUnit: USD Stage 的单位缩放因子，用于将局部单位转换为米

## 关联笔记
- 01KJC0839D1FRD3FGTXZ5677H2.md: 同样涉及 Isaac Sim 中的坐标系转换，讨论 OpenGL 与 ROS 光学坐标系的差异及 TF 变换链
