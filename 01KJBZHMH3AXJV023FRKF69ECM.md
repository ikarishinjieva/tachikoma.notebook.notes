---
title: 20241022 - 分子识别: OSRA 实验+论文
confluence_page_id: 3342777
created_at: 2024-10-22T14:55:00+00:00
updated_at: 2024-10-22T15:08:54+00:00
---

# 测试

图形: ![e25c2804bffc45323ebb08716f22467c1466991baf6f2a768ffc1ed586173ac8.png](/assets/01KJBZHMH3AXJV023FRKF69ECM/e25c2804bffc45323ebb08716f22467c1466991baf6f2a768ffc1ed586173ac8.png)

该图形通过大模型识别效果不好

通过OSRA测试: <https://cactus.nci.nih.gov/cgi-bin/osra/index.cgi>

![image2024-10-22 22:49:29.png](/assets/01KJBZHMH3AXJV023FRKF69ECM/image2024-10-22%2022%3A49%3A29.png)

生成的structure很接近

# 论文

```
这篇论文描述了OSRA，一个开源的光学结构识别软件，用于从文档中提取化学结构信息。其叙事结构如下：

1. **引言 (Introduction):**  点明化学文献中存在大量以图片形式存在的化学结构信息，而这些信息难以被计算机直接处理。现有软件方案未能完美解决这个问题，因此需要一个开源的解决方案。OSRA应运而生。

2. **算法 (Algorithm):** 这是论文的核心部分，详细讲解了OSRA的图像处理算法流程，包含以下步骤：
    * 灰度化和二值化 (Grayscale and Binarization)
    * 分割 (Segmentation)
    * 各向异性平滑和细化 (Anisotropic smoothing and thinning)
    * 矢量化和键/节点检测 (Vectorization and bond/node detection)
    * 原子标签和电荷识别 (Atomic label and charge recognition)
    * 圆键识别 (Circle bond recognition)
    * 双键和三键检测 (Double and triple bond detection)
    * 特殊键检测：楔形键和虚线键 (Special bond detection: wedge and dash bonds)
    * 桥键检测 (Bridge bond detection)
    * 连接表编译 (Compilation of the connection table)
    * 置信度估计 (Confidence estimate)

3. **讨论 (Discussion):**  讨论了OSRA的性能评估方法，提出了使用Tanimoto相似性指数作为衡量标准，并与市面上唯一的商业软件CLiDE进行了比较。 提供了在小型测试集和内部测试集上的测试结果，并分析了主要的错误来源。最后，强调了开源的意义和未来的发展方向。

4. **致谢 (Acknowledgments):** 感谢资助机构。

总而言之，论文以“问题-方案-方法-结果-展望”的逻辑展开，清晰地介绍了OSRA的功能、实现方法和性能评估。
``` 

```
这篇论文详细描述了 OSRA 用于光学化学结构识别的图像处理算法，包含以下步骤，我将分别解释每一步是如何进行的：

1. **灰度化和二值化 (Grayscale and Binarization):**
    * **灰度化:**  彩色图像首先转换为灰度图像。不同于常用的灰度转换方法 `Gr = (R + G + B)/3`，OSRA 使用 `Gr = min(R, G, B)`，目的是为了在后续的二值化步骤中更好地处理图像中浅色的部分（例如硫的黄色符号）。
    * **二值化:** 使用全局阈值将灰度图像转换为黑白图像。虽然也测试过局部（自适应）阈值，但由于阈值变化区域中会出现伪影，效果并不理想。
    * **多尺度处理:**  图像会以三种不同的分辨率（72、150 和 300 dpi）进行处理，除非是 PDF 或 Postscript 文档，那样只使用 150 dpi。分辨率会影响最大字符大小和整体分子图像大小的限制，以及细化和平滑算法的选择。

2. **分割 (Segmentation):**
    * 从原始图像中选择包含化学结构的矩形区域。选择的标准包括：
        * 黑色像素与矩形总面积之比在 0.0 到 0.2 之间。
        * 长宽比在 0.2 到 5.0 之间。
        * 矩形不与现有的包含结构的矩形相交。
        * 如果分辨率高于 150 dpi，宽度和高度必须大于最小值（目前为 50 像素）。
        * 缩放至 300 dpi 后的宽度和高度小于最大值 1000 像素（如果分辨率高于 150 dpi）。

3. **各向异性平滑和细化 (Anisotropic smoothing and thinning):**
    * **噪声因子计算:**  对于每个分割出的矩形区域，计算“噪声因子”，定义为长度为 2 像素的线段数量与长度为 3 像素的线段数量之比。
    * **各向异性平滑:** 如果图像噪声过大（噪声因子在 0.5 到 1.0 之间），则执行各向异性平滑。使用 GREYCstoration 各向异性平滑库去除小的像素强度变化，同时保留全局图像特征，该库基于非线性多值扩散偏微分方程。
    * **细化:** 使用细化函数将所有线条标准化为 1 像素宽，使用 Joseph M. Cychosz 的文章 “Efficient Binary Image Thinning using Neighborhood Maps” 中的子程序。
    * **分辨率限制:** 目前，各向异性平滑和细化仅对分辨率为 300 dpi 的图像执行。

4. **矢量化和键/节点检测 (Vectorization and bond/node detection):**
    * **矢量化:** 使用 Potrace 库将位图转换为矢量图形。
    * **原子和键识别:** 通过检查 Bezier 曲线的控制点之间的间隔来找到原子和键的位置。一个控制点被标记为原子的条件包括：Potrace 算法将其归类为角点；与前一个键相比，该点到下一个控制点的向量表示方向的显著变化；该点到下一个控制点的距离小于前一个原子到该点的距离。键被设置为连接找到的原子的向量。
    * **法向分量:** 使用法向分量而不是两个向量之间的角度来判断，这样更稳健。

5. **原子标签和电荷识别 (Atomic label and charge recognition):**
    * 使用 GOCR 和 OCRAD（开源 OCR 工具）识别小于最大字符高度/宽度或水平/垂直排列的两个字符的连通 Bezier 曲线集，识别其是否为杂原子标签或缩写。
    * 去除已识别字符对应的 Bezier 曲线。
    * 识别并分配形式电荷（“+”和“-”）。

6. **圆键识别 (Circle bond recognition):**
    * 如果在环内找到足够大直径的圆，则将该环标记为芳香环。其他条件包括：环原子足够靠近圆；环键与指向圆心的向量之间的角度小于 90 度。

7. **双键和三键检测 (Double and triple bond detection):**
    * 估算平均键长（使用排序后键长列表的 75% 分位数）。
    * 识别彼此平行、距离足够近且位于彼此“阴影”内的键对（或三键）。

8. **虚线键和楔形键检测 (Dashed and wedge bonds):**
    * **虚线键:** 识别为三个或更多的小“斑点”，这些斑点位于一条直线的平均键长范围内。
    * **楔形键:** 通过构建键厚度与位置的线性回归来识别，并测试沿键的厚度是否存在显著增加或减少。

9. **桥键检测 (Bridge bond detection):**
    * 如果一个原子连接到四个成对共线的单键（都不是末端键），并且移除该原子节点不会导致片段数量、可旋转键数量的变化，或 5 元环和 6 元环的数量减少 2 个，则移除该原子，并将该交叉点推断为桥键交叉点。

10. **连接表编译 (Compilation of the connection table):**
    * 使用 OpenBabel 或 RDKit 构建分子对象，包含连接性、立体化学和芳香性信息。
    * 添加基于超原子的片段。

11. **置信度估计 (Confidence estimate):**
    * 使用一个置信度函数，该函数基于元素计数、环计数和片段数量等简单的分子属性，通过线性回归分析得出。用于在不同分辨率下生成的多个结果中选择最佳结果。

希望以上解释能够帮助您理解 OSRA 图像处理的每一步。
```
