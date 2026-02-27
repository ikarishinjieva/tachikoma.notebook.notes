---
title: 20250203 - 阅读论文: LayoutReader: Pre-training of Text and Layout for Reading Order Detection
confluence_page_id: 3344124
created_at: 2025-02-03T03:46:58+00:00
updated_at: 2025-02-03T03:46:58+00:00
---

# 有用的思路 

论文主要想训练一个模型, 识别一个PDF的阅读顺序. 

其中有用的思路是: 如何创建训练数据: "为了解决数据匮乏的问题，作者提出了ReadingBank，一个包含50万文档图像的大规模阅读顺序检测基准数据集。该数据集利用WORD文档的XML元数据中嵌入的阅读顺序信息，并将其与从WORD转换而来的PDF或图像中的布局信息对齐，从而自动生成标注"

XML中的阅读顺序信息, 和人类的阅读习惯可能有所不同, 但大部分情况是类似的
