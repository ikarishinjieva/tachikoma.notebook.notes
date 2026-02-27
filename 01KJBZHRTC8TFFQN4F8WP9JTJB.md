---
title: 20241023 - 对osra 1.4.0的代码解析
confluence_page_id: 3342791
created_at: 2024-10-23T06:51:54+00:00
updated_at: 2024-10-23T08:36:49+00:00
---

- 加载拼写修正表: spelling.txt
  - 加载超级原子的映射表: superatom.txt
    - 超级原子是SMILE表达式中规定的一些原子团的代号系统
  - 用GraphicsMagick加载图片
  - 将图片转成灰度: convert_to_gray
  - 处理图片的解析度resolution
  - 处理图片的旋转
  - 解析PDF的表格 (unpaper)
  - 识别出图片的中的clusters (?)
  - clusters拼接成box
  - 用多种分辨率 (select_resolution) 反复进行如下步骤: 
    - 逐个box处理:
      - 将位图转为矢量图
      - 通过对SVG矢量的遍历, 获取原子和键的列表 (find_atoms + find_bonds)
      - find_chars
      - ... (后面都是类似的图像处理)
