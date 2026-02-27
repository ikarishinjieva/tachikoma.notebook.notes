---
title: 20250526 - SQLShift - 尝试对存储过程进行分区转换
confluence_page_id: 3997755
created_at: 2025-05-27T06:13:18+00:00
updated_at: 2025-05-27T06:13:18+00:00
---

让AI生成推荐方案: 

  - <https://poe.com/s/KuU9i8BzybG30k3szAl4>
  - <https://poe.com/s/KlAXCZLwQuvHFccULRjc>

# 最终使用的方案

  - <https://poe.com/s/KlAXCZLwQuvHFccULRjc>
  - 简单描述: 
    - 使用给模型几百行片段 + 上文总结, 让模型: 
      - 先进行分块
      - 对分块进行改写
      - 拼接改写后的片段
      - 将完整的存储过程进行最终检查

已移交给虎青
