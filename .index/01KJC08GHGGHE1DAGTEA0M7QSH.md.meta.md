---
note: 01KJC08GHGGHE1DAGTEA0M7QSH.md
title: 20251105 - 在Isaac Sim中, 获取选择Mesh的世界系坐标
indexed_at: 2026-02-27T11:52:51.179689+00:00
---

用户要求我分析一篇关于 Isaac Sim 中获取 Mesh 世界坐标的笔记，并生成索引元数据。让我分析笔记内容：

1. 笔记主题：在 Isaac Sim 中获取选中 Mesh 的世界坐标系坐标
2. 核心内容：Python 脚本，包含矩阵转换、坐标变换、SVD 分解等功能
3. 技术栈：Isaac Sim、USD (Universal Scene Description)、Python、numpy

标签应该包括：Isaac Sim、坐标变换、USD、Python、Mesh、世界坐标等

摘要需要概括这是用于在 Isaac Sim 中获取选中 Mesh 的世界坐标、旋转和尺寸的脚本

关键概念：
- Gf.Matrix4d: USD 的 4x4 矩阵类型
- XformCache: USD 的变换缓存
- AABB: 轴对齐包围盒
- SVD 分解: 用于从变换矩阵提取旋转和缩放

关联笔记：查看提供的笔记列表，都是关于 MySQL、Clickhouse、数据库、性能调优等内容，与 Isaac Sim/机器人仿真无关，所以应该是「无」## 标签
Isaac Sim, 坐标变换, USD, Python, Mesh, 世界坐标

## 摘要
本文记录了在 Isaac Sim 中获取选中 Mesh 世界坐标系坐标的 Python 脚本。包含变换矩阵分解、局部坐标到世界坐标转换、以及 Mesh 世界尺寸计算等功能。

## 关键概念
- Gf.Matrix4d: USD 的 4x4 齐次变换矩阵类型
- UsdGeom.XformCache: USD 用于缓存和计算几何变换的类
- AABB: 轴对齐包围盒，用于计算 Mesh 的世界尺寸
- SVD 分解: 从变换矩阵中分离平移、旋转、缩放分量

## 关联笔记
无
