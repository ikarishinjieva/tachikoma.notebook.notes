---
title: 20240607 - 阅读论文: FiD-Light: EFFICIENT AND EFFECTIVE RETRIEVAL-AUGMENTED TEXT GENERATION
confluence_page_id: 2949602
created_at: 2024-06-07T07:43:46+00:00
updated_at: 2024-06-07T08:38:34+00:00
---

# FiD架构

主要思路: 将问题和召回文档进行向量编码, 将这个长向量 放到 解码器中, 生成答案. 相当于: 在解码过程中进行思考

# FiD-Light

在Fid的解码阶段之前, 对长向量进行压缩, 减少解码成本

# Source Pointers

在向量中, 增加源文档的标记, 在解码器中保留标记, 做到可以对输出进行示踪

![image2024-6-7 16:38:30.png](/assets/01KJBZ9RYTBE5YJST0JVNP925P/image2024-6-7%2016%3A38%3A30.png)
