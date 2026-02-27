---
title: 20240729 - 阅读论文: Move Beyond Triples: Contextual Knowledge Graph Representation and Reasoning
confluence_page_id: 3145897
created_at: 2024-07-29T03:40:18+00:00
updated_at: 2024-07-29T05:26:52+00:00
---

# 问题定义

在传统知识图谱中, 由于实体和关系没有"上下文"信息, 导致在不同的上下文中, 会发生冲突, 举例: 

![image2024-7-29 11:31:1.png](/assets/01KJBZBQRPWGJG3WG5BDBV0Y2Y/image2024-7-29%2011%3A31%3A1.png)

# 解决方案

将上下文放到图谱中, 比如放在属性图谱的属性中 (将 KG 中的结构化信息 (三元组) 和非结构化信息 (上下文) 结合起来)

搜索步骤: KGE 模型负责初步的候选答案检索和排序，LLM 则利用上下文信息对候选答案进行进一步的排序和推理。
