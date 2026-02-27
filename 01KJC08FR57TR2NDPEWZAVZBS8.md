---
title: 20251105 - 在Isaac Sim中调整视角
confluence_page_id: 4359051
created_at: 2025-11-05T05:55:30+00:00
updated_at: 2025-11-05T05:55:30+00:00
---

看起来是视口相机“掉到地下/地平线以下”了。常用的几种快速恢复与操作方式如下（适用于 Omniverse/Isaac Sim 视图窗口）：

快速脱困

  - F 聚焦到选中物体
    - 在 Stage 里点选任意可见的物体（比如杯子或地面），鼠标停在视口内，按 F。视口相机会飞到合适距离并对准它。
  - A 框选聚焦
    - 按 A（Frame All），把场景所有物体框入视野，常能把相机拉回地面以上。
  - 重置相机位姿
    - 视口右上角相机菜单（通常显示 Perspective）> Reset Camera 或选择已保存的相机书签。
  - 切换为正交视图再回来
    - 按空格键（或点击视口右上角的方形 gizmo 的轴向）切到 Top/Front/Right 等正交视图，再切回 Perspective。

常用相机导航键（默认）

  - 右键 + WASD：自由飞行（Q/E 下/上升）
    - Shift 加速，鼠标右键转向，滚轮调速度（或相机速度滑条）。
  - 中键拖拽：平移
  - 右键拖拽：环视
  - Alt + 左键拖：轨道环绕
  - Alt + 中键拖：平移
  - Alt + 右键拖或滚轮：缩放/推拉
  - G：切换地面吸附（Toggle Ground Plane Snap）。若开启，“飞行”常保持在地面附近。
  - Ctrl/Cmd + 1/2/3...：保存/调用相机书签（Bookmarks > Create Bookmark）。出问题时一键返回。

避免再次“掉地下”的设置

  - 启用相机碰撞或地面限制
    - 在 Navigation/Viewport 设置里将相机与场景碰撞开启（某些版本提供 Camera Collision/Snap to Ground），避免穿透地面。
  - 调整相机移动速度
    - 视口右上角相机图标附近有速度滑块；把速度调小，避免滚轮一下子穿越地面。
  - 使用固定相机
    - 在 /World 下创建一个 Camera Prim，固定位置；需要时右上角相机菜单切到该相机视图。

如果上述快捷键无效，可能你当前焦点不在视口内或键位被修改。你可以在：

  - Edit > Preferences > Navigation（或 Viewport）里恢复默认键位。
  - 视口右上角“汉堡菜单”里 Reset Viewport Settings。

想更精确一点的话，你也可以直接选中“当前视口相机”并数值编辑其变换：

  - 右上角选择一个具体 Camera 视图（不是 Perspective），在 Property 里把 Z 高度调高，比如 Z=1.5 m；旋转 Pitch 向下俯视约 -15° 到 -30°。

需要的话告诉我你用的是哪个版本的 Isaac Sim/Omniverse，我可以给对应版本的具体菜单路径。
