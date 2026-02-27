---
title: 20250130 - "拒绝采样"的概念
confluence_page_id: 3344037
created_at: 2025-01-30T15:45:48+00:00
updated_at: 2025-03-17T16:01:28+00:00
---

**<https://mp.weixin.qq.com/s/iRFulAZVs_tBVX5R5dYZlA>**

**  
**

**我们来看看Llama做Post-training的飞轮过程：**

1.持续通过人工标注或机造方式富集偏好pair样本，训练Reward Model

2.基于当前能力最好的模型，随机采集一批 ，每个Prompt拿最好的模型做  次数据生成采样，每个Prompt就得到 条  数据

3.拒绝采样：对第2步采样 个 数据，用Reward Model打分，并从中选取打分最高 topN 条样本。作为指令微调的精选样本，训练SFT Model。

4.训完SFT Model，再通过持续收集的偏好对样本（同步骤1）做对齐学习（Llama使用的是DPO）。最终得到了一个比当前模型更好的模型

5.持续做步骤1~步骤4，飞轮迭代优化模型。

拒绝采样是我们自动化做样本工程的非常重要方法，这里面Reward Model 是最关键的角色。

  

  

![image2025-3-18 0:1:22.png](/assets/01KJBZQXDMK6QKQ7VMPCJQ4G4T/image2025-3-18%200%3A1%3A22.png)
