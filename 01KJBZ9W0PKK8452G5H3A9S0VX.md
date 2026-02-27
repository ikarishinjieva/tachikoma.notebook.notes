---
title: 20240613 - 阅读论文: FLAR: Active Retrieval Augmented Generation
confluence_page_id: 2949673
created_at: 2024-06-13T07:09:57+00:00
updated_at: 2024-06-13T07:09:57+00:00
---

# 主要思路

  - 在LLM进行推理时, 主动进行进一步的检索, 以完善答案
  - 何时进行进一步检索:
    - FLAREinstruct: 在LLM推理时, 直接生成检索鼓励指令
      - 举例:   
  
![image2024-6-13 15:8:16.png](/assets/01KJBZ9W0PKK8452G5H3A9S0VX/image2024-6-13%2015%3A8%3A16.png)
    - FLAREdirect: 在出现不确定的token时 (token的置信概率小于阈值), 进行主动检索来替换该句子
