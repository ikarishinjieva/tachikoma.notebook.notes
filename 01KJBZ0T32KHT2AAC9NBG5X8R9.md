---
title: 20230608 - Text-To-SQL 的测试基准集 和 UnifiedSKG框架
confluence_page_id: 2392126
created_at: 2023-06-08T04:25:44+00:00
updated_at: 2023-06-12T10:07:32+00:00
---

# DAMO-ConvAI 测试基准集

  - <https://github.com/AlibabaResearch/DAMO-ConvAI/tree/main/bird>
    - 其作用是 测试 text-to-SQL 的准确度
    - 其中有fine tuning方法, 使用 <https://github.com/HKUNLP/UnifiedSKG> 的方法, 调优 T5模型
  - 数据集下载: <https://drive.google.com/drive/folders/1GXigUv3MU-Sh4XiY6Wz3xVeNT_s0SuON>
    - 包含SQL的数据集: cosql/sparc/spider/sql2text/wikisql

# UnifiedSKG 模型处理框架

  - Structured knowledge grounding (SKG) 是 结构化知识 (SQL/表格等)
  - UnifiedSKG 融合了 21个预处理任务, 将结构化知识变成文本, 由文本大模型进行理解和处理

![image2023-6-8 13:5:38.png](/assets/01KJBZ0T32KHT2AAC9NBG5X8R9/image2023-6-8%2013%3A5%3A38.png)

# 待处理的问题

  - 如果想解决SQLe的自定义SQL规则问题, 需要一个能理解SQL的模型, 以及常见的规则描述
