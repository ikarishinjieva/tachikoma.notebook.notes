---
title: 20230614 - 对t5进行微调, 输入sql审核规则
confluence_page_id: 2392160
created_at: 2023-06-15T00:47:08+00:00
updated_at: 2023-06-15T00:47:08+00:00
---

# notebook

[附件: Untitled.ipynb] 

# 效果

使用mT5模型, 进行微调. 

发现其对大小写敏感: 

![image2023-6-14 1:7:3.png](/assets/01KJBZ1946E5HSTVJ65CH42XRH/image2023-6-14%201%3A7%3A3.png)

![image2023-6-15 8:42:11.png](/assets/01KJBZ1946E5HSTVJ65CH42XRH/image2023-6-15%208%3A42%3A11.png)

将提示词反义, 结果仍然不变: 

![image2023-6-15 8:42:47.png](/assets/01KJBZ1946E5HSTVJ65CH42XRH/image2023-6-15%208%3A42%3A47.png)

怀疑T5-base模型对于语言理解并不好, 导致如果没有大量英文语料逻辑, 对于rule的自然语言书写不易被理解

# 测试T5-base的语言理解能力

![image2023-6-15 8:44:8.png](/assets/01KJBZ1946E5HSTVJ65CH42XRH/image2023-6-15%208%3A44%3A8.png)

逻辑理解确实会差
