---
title: 20240726 - 阅读论文: Retrieval-Augmented Generation with Knowledge Graphs for Customer Service Question Answering
confluence_page_id: 3145888
created_at: 2024-07-26T11:48:56+00:00
updated_at: 2024-07-26T11:52:33+00:00
---

创新点在于构架图谱的方案: 

使用了两级结构: 

  1. 工单内部, 各个章节之间建立联系
  2. 各个工单之间, 建立联系

![image2024-7-26 19:48:22.png](/assets/01KJBZBNDVXWSRG9XQMH1EC20A/image2024-7-26%2019%3A48%3A22.png)

样例: 

![image2024-7-26 19:50:38.png](/assets/01KJBZBNDVXWSRG9XQMH1EC20A/image2024-7-26%2019%3A50%3A38.png)

注意: 

其问题都是跟"属性" (ticket的属性)有关, 所以第一级关系的建立是将 工单的各部分连接起来

(也就是工单其实是结构化数据, 按照结构建立了第一层关系)
