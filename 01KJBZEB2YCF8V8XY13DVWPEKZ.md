---
title: 20240815 - 阅读论文*: Learning to Generate Prompts for Dialogue Generation through Reinforcement Learning
confluence_page_id: 3146162
created_at: 2024-08-15T09:30:00+00:00
updated_at: 2024-08-15T10:09:30+00:00
---

# 主要思路

微调大模型的成本过高 (需要大量数据, 对于在线模型的训练代价较高)

通过 训练一个小模型A, 根据输入, 让A生成提示词P, 用P去调用大模型, 生成输出. 

这样只需要训练小模型A (强化学习算法), 就可以改变大模型的行为
