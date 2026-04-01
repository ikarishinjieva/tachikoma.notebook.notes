---
title: 20260401 - 富乐华 CAD 识别
created_at: 2026-04-01T02:31:04.987468+00:00
updated_at: 2026-04-01T02:36:16.647123+00:00
---

# 需求

对CAD中的长度进行校验

# 仓库

<https://github.com/ikarishinjieva/fulehua.ai>

# 经验

放在了 requirement/4\_[CAD复核问题与经验总结.md](http://CAD复核问题与经验总结.md)

重点: 

- 想在界面上完好显示CAD, 得 用 `ezdxf.addons.drawing` 直接从 DXF 生成 SVG. 不能自绘CAD, 效果差很多
- 其尺寸标注和辅助线 都在 DIMENSION 对象中, 但序号不是DIMENSION对象中, 得单独去匹配. 匹配算法得反复调整
- 比例尺 子图的识别: 有一些子图有比例尺, 现在是识别出哪些是比例放大的组件, 然后将其归纳到子图, 再校验其闭包内没有其他子图
- “形位公差”, 是一个特殊的对象, 从不同CAD软件 转换出来的 对象不一样