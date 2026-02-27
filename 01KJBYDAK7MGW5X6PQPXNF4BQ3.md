---
title: 20210429 - SQLLancer 学习
confluence_page_id: 754049
created_at: 2021-04-29T09:28:28+00:00
updated_at: 2021-04-29T09:28:28+00:00
---

# 资料

<https://heisenbug-piter.ru/en/2021/spb/talks/nr1cwknssdodjkqgzsbvh/>

# 摘要

  - SQLLancer主要分析SQL处理的逻辑错误
  - 目前使用三种逻辑推断方法: 
    - ![image2021-4-29 16:50:44.png](/assets/01KJBYDAK7MGW5X6PQPXNF4BQ3/image2021-4-29%2016%3A50%3A44.png)
  - Ternary Logic Partitioning: 相当于Query Partition

    - ![image2021-4-29 17:1:44.png](/assets/01KJBYDAK7MGW5X6PQPXNF4BQ3/image2021-4-29%2017%3A1%3A44.png)
    - 将Query拆分成多个子Query, 将结果集合并, 与拆分之前的结果集进行对比
    - 举例: 
      - ![image2021-4-29 17:3:16.png](/assets/01KJBYDAK7MGW5X6PQPXNF4BQ3/image2021-4-29%2017%3A3%3A16.png)
      - ![image2021-4-29 17:3:19.png](/assets/01KJBYDAK7MGW5X6PQPXNF4BQ3/image2021-4-29%2017%3A3%3A19.png)
  - Non-optimizing Reference Engine Construction: 测试 优化前 和 优化后 的SQL的区别, 针对于优化器的正确性进行测试
    - ![image2021-4-29 17:5:34.png](/assets/01KJBYDAK7MGW5X6PQPXNF4BQ3/image2021-4-29%2017%3A5%3A34.png)
    - 举例: 
      - ![image2021-4-29 17:6:11.png](/assets/01KJBYDAK7MGW5X6PQPXNF4BQ3/image2021-4-29%2017%3A6%3A11.png)
  - Pivoted Query Synthesis: 
    - 随机选取一行, 构造条件, 下SQL检查结果中是否包括该行
  - 三种方法比较: 
    - ![image2021-4-29 17:24:57.png](/assets/01KJBYDAK7MGW5X6PQPXNF4BQ3/image2021-4-29%2017%3A24%3A57.png)
