---
title: 20240615 - 阅读论文: TIARA: Multi-grained Retrieval for Robust Question Answering over Large Knowledge Bases
confluence_page_id: 2949716
created_at: 2024-06-15T15:06:20+00:00
updated_at: 2024-06-15T15:06:20+00:00
---

主要思路: 

![image2024-6-15 22:46:54.png](/assets/01KJBZA42C3TD742BTWQV5XFNZ/image2024-6-15%2022%3A46%3A54.png)

论文要解决SQL生成的问题 (结构化数据生成), 使用了以下几种组件: 

  1. 实体识别 (Entity Retrieval)
     1. 先抽取实体信息 (Mention Detection)
     2. 然后选取候选的实体 (Candidate Generation), 其中 "m.0121_ " 是 "napa valley" 的主键
     3. 最后从候选实体中, 选取合适的实体 (Entity Disambiguation)
  2. 查找形式逻辑 (根据实体识别的结果) (Exemplary Logical From Retrieval)
     1. 从实体触发, 找到两跳内的所有关系 (从训练数据库中)
  3. 查找Schema (这是个独立的步骤) (Schema Retrieval)
     1. 查找 跟问题相关的所有 Class (表名), 和 Relation (表连接关系)
  4. 进行生成: 
     1. 在生成时, 需要模型学习Schema查找的结果 (表名和表关系), 仅在这些结果中进行SQL构建, 这样会减少伪造 (Decoding Constraints)
