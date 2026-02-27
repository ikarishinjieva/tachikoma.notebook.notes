---
title: 20230503 - 如何使用图片作为LLM的上下文 (暂停)
confluence_page_id: 2130850
created_at: 2023-05-03T07:48:31+00:00
updated_at: 2023-05-05T09:14:28+00:00
---

# 测试数据

![image2022-4-22 23_14_36.png](/assets/01KJBYZ45PDZJEXX7NNXHHGJY4/image2022-4-22%2023_14_36.png)

# 测试unstructured方案

<https://python.langchain.com/en/latest/ecosystem/unstructured.html>

用容器启动unstructured api, 识别图片: 

```
curl -X 'POST' \
>   'http://10.186.16.136:8000/general/v0/general' \
>   -H 'accept: application/json' \
>   -H 'Content-Type: multipart/form-data' \
>   -F 'files=@/tmp/1.png' \
>   -F 'strategy=hi_res' \
>   | jq -C . | less -R
``` 

结果: 不能很好的识别全部文字

```
[
  {
    "element_id": "8c24ab12798464a40eaa7e629ec269fb",
    "text": "table_name=\"' ‘SYS_TABLES‘' ,\n\n1 row in set (45.48 sec)\nmysql> [\n",
    "type": "FigureCaption",
    "metadata": {
      "page_number": 1,
      "filename": "1.png"
    }
  },
  {
    "element_id": "e910f529d7fbc7209d808c6830148987",
    "text": "nld:-..innodb_bnt!u _Page where tnblo_nm-\"m_nlm\"\n\nlltiu_m.hnodb_buzto: Page where table_name=' ‘SYS_TABLES‘' angd OLDEST |\n\n",
    "type": "FigureCaption",
    "metadata": {
      "page_number": 1,
      "filename": "1.png"
    }
  }
]
``` 

# 测试 Tesseract OCR

<https://cdkm.com/ocr>

识别结果: 

```
| count(1) )|
R

| 1252099 |

1 row in set (45.48 sec)

 

mysql> [
``` 

# 测试 Tesseract OCR + Pillow图片增强

```
import pytesseract
from PIL import Image, ImageFilter, ImageEnhance

# 读取图片并进行预处理
img = Image.open('1.png')
img = img.filter(ImageFilter.SHARPEN)
img = img.convert('L')
img = ImageEnhance.Contrast(img).enhance(2)
img = ImageEnhance.Brightness(img).enhance(1.5)

# 使用Tesseract OCR进行文本识别
result = pytesseract.image_to_string(img, lang='eng')

print(result)
``` 

结果: 

```
i:
Waren nnn ny

-innodb buffer sty be ee ST TTT Ty pT.

Co eee ee page where At “SY8_TABLES** and feo ha 7-5 Jo's

4.

a
1252099 |

—————— yp

1 row in set (45.48 r TY}

P Lertee |

Ss

``` 

# 测试paddle ocr

<https://www.paddlepaddle.org.cn/hub/scene/ocr>

识别结果:

![image2023-5-4 0:18:53.png](/assets/01KJBYZ45PDZJEXX7NNXHHGJY4/image2023-5-4%200%3A18%3A53.png)

# 测试 Tesseract OCR + scikit-image图片增强

将所有中间文件输出, 判断哪个步骤的效果欠佳

```
import numpy as np
from PIL import Image
import skimage.io as io
from skimage.color import rgb2gray
from skimage.filters import threshold_otsu
from skimage.transform import rotate, resize
from skimage import exposure
import pytesseract

# 读取图像
image = io.imread('1.png')
# 仅保留 RGB 通道
image = image[:, :, :3]

# 保存原始图像
io.imsave('01_original.png', image)

# 对 RGB 图像进行对比度增强，以加强文本部分
p2, p98 = np.percentile(image, (2, 98))
image = exposure.rescale_intensity(image, in_range=(p2, p98))

# 将 RGB 图像转换为 PIL 图像对象，并保存
original_pil = Image.fromarray(np.uint8(image))
original_pil.save('02_contrast.png')

# 转换为灰度图像
gray_image = rgb2gray(image)

# 对灰度图像进行对比度增强，以减轻白色覆盖效果
p2, p98 = np.percentile(gray_image, (2, 98))
gray_image = exposure.rescale_intensity(gray_image, in_range=(p2, p98))

# 将灰度图像转换为 PIL 图像对象，并保存
gray_pil = Image.fromarray(np.uint8(gray_image * 255))
gray_pil.save('03_gray.png')

# 应用 Otsu 二值化算法
threshold_value = threshold_otsu(gray_image)
binary_image = gray_image > threshold_value

# 将二值化图像转换为 PIL 图像对象，并保存
binary_pil = Image.fromarray(np.uint8(binary_image * 255))
binary_pil.save('04_binary.png')

# 旋转图像，使其水平
angle = 0
edges = None
for i in range(-10, 10):
    rotated = rotate(binary_image, i, resize=True)
    if edges is None or len(np.where(rotated)[0]) > len(np.where(edges)[0]):
        angle = i
        edges = rotated

# 将旋转后的图像转换为 PIL 图像对象，并保存
rotated_pil = Image.fromarray(np.uint8(edges * 255))
rotated_pil.save('05_rotated.png')

# 调整亮度和对比度
adjusted_image = exposure.rescale_intensity(edges, in_range=(0.4, 0.6))

# 将调整后的图像转换为 PIL 图像对象，并保存
adjusted_pil = Image.fromarray(np.uint8(adjusted_image * 255))
adjusted_pil.save('06_adjusted.png')

# 使用 Pytesseract 进行 OCR
text = pytesseract.image_to_string(adjusted_image, lang='eng')

# 输出识别结果
print(text)
``` 

发现在二值化的结果中, 将显示器的反光识别成留白. 需要调整二值化的阈值. 

探索各种自适应阈值的函数效果: 

```
#在jupyter中运行
!pip install scikit-image -i https://pypi.tuna.tsinghua.edu.cn/simple
import skimage.io as io
from skimage.filters import try_all_threshold

# 读取图像
image = io.imread('image2022-4-22 23_14_36.png', as_gray=True)

# 测试不同阈值方法的效果
fig, ax = try_all_threshold(image, figsize=(10, 8), verbose=False)

# 显示结果
#plt.show()
``` 

效果: 

![image]()

# TODO

OCR（Optical Character Recognition）库是用于将图像或扫描文档中的文字转换为可编辑文本的工具。以下是几个常用的OCR库：

  1. Tesseract OCR：由Google开发，是目前最常用的OCR库之一，支持多种语言，并提供预先训练的模型。可用于C++，Python，Java等多种编程语言。

  2. OCRopus：一款基于Python的OCR引擎，具有模块化设计，易于扩展。它包含许多优秀的预处理算法和多种语言的OCR模型。

  3. Kraken：一个OCR引擎，旨在提供一种灵活而可定制的OCR工作流程。它基于Python并使用深度学习模型，支持多种语言。

  4. GOCR：另一个开源OCR库，具有多种语言支持，并使用了一些预处理算法，如二值化和降噪。

  5. Microsoft OCR：Microsoft的计算机视觉API提供了一个OCR API，可以在Azure云上运行，支持多种语言和图像格式。

这些OCR库都具有其独特的特点和优点，选择哪一个取决于你的需求和项目的特定要求。

\---

低分辨率、模糊或歪斜的文本识别是OCR领域的难点问题之一。以下是一些OCR库或云服务，在处理这些困难情况时比较出色：

  1. Tesseract OCR：Tesseract OCR在低分辨率、模糊或歪斜的文本识别方面具有一定的优势，特别是在最新版本的4.1.0中，对于低分辨率和模糊的图像有了很大的改进。可以通过对图像进行预处理（如二值化、去噪等），来进一步提高识别精度。

  2. Kraken：Kraken使用基于深度学习的方法来处理低分辨率、模糊或歪斜的文本，这使得它在这些方面表现比较出色。Kraken还可以与Tesseract OCR结合使用，以进一步提高识别精度。

  3. Microsoft OCR：Microsoft OCR的文本识别模型使用了深度卷积神经网络，并针对低分辨率、模糊或歪斜的文本进行了优化，因此在这些方面表现相对出色。此外，Microsoft OCR还支持多种语言和图像格式，并提供了API接口，方便快捷。

需要注意的是，即使这些OCR库或云服务在低分辨率、模糊或歪斜的文本识别方面表现较好，识别精度仍然可能受到多种因素的影响。因此，在使用这些OCR库或云服务时，建议进行适当的预处理和后处理，以进一步提高识别精度。
