---
title: 20250103 - 阅读论文: KeyInst: Keyword Instruction for Improving SQL Formulation in Text-to-SQL
confluence_page_id: 3343699
created_at: 2025-01-03T12:14:54+00:00
updated_at: 2025-01-03T12:14:54+00:00
---

# 有用的知识

主要思路: KeyInst 对 输入进行思考, 要求给出 合适的SQL关键字 (Join/Order by等)

举例: 

![image2025-1-3 20:14:19.png](/assets/01KJBZPTT8A443T3Q5V8SWX4QJ/image2025-1-3%2020%3A14%3A19.png)

# 局限
    
    
    KeyInst 主要针对的是**关键 SQL 关键字代表的逻辑缺失或误用**这一类错误. 但Text2SQL中, 这一类的错误是否足够多, 不清楚

```
Text2SQL 的常见错误多种多样，但**关键 SQL 关键字代表的逻辑缺失或误用确实是其中一类重要错误，并且往往会导致查询结果完全错误**。  除了这种错误，其他类型的错误也同样值得关注，例如：

1. **Schema Linking 错误:**  这是 Text2SQL 中最常见的错误类型之一，指 LLM 没有正确地将自然语言中的实体或概念映射到数据库 schema 中的表和列。例如，NLQ 中提到的 "customer name" 应该对应数据库中的 `customers.name` 列，但 LLM 可能会错误地链接到 `orders.customer_id` 列。这类错误即使 SQL 结构正确，也会导致查询结果错误或无意义。

2. **值匹配错误:** LLM 可能无法正确识别 NLQ 中提到的值，或者将其转换为 SQL 中合适的格式。例如，NLQ 中的日期 "January 1st, 2024"  需要转换为 SQL 中的日期格式 `'2024-01-01'`，如果转换错误，就会导致查询失败或结果错误。

3. **条件逻辑错误:**  即使 LLM 正确识别了关键词和 schema 元素，也可能无法正确理解 NLQ 中的条件逻辑，例如 `AND`, `OR`, `NOT` 等逻辑运算符的用法。这会导致生成的 SQL 中的 `WHERE` 子句不符合 NLQ 的语义。

4. **聚合函数误用:**  LLM 可能会误用或遗漏聚合函数，例如 `COUNT`, `SUM`, `AVG`, `MAX`, `MIN` 等。例如，NLQ 要求计算某个值的平均值，但 LLM 生成的 SQL 却计算了总和。

5. **嵌套和子查询错误:**  对于复杂的 NLQ，LLM 可能会在嵌套子查询的结构或逻辑上出错，导致生成的 SQL 语法错误或语义错误。

6. **数据类型不匹配:** LLM 生成的 SQL 中的数据类型可能与数据库 schema 中的列类型不匹配，导致查询错误。

KeyInst 主要针对的是**关键 SQL 关键字代表的逻辑缺失或误用**这一类错误，特别是那些高优先级的关键词，例如 `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`, `UNION`, `INTERSECT`, `EXCEPT` 和 `WHERE`。这些关键词通常直接决定了 SQL 查询的核心逻辑，如果缺失或误用，就会导致查询结果完全错误。  KeyInst 通过显式地提醒 LLM 注意这些关键词，可以有效降低这类错误的发生概率。

然而，KeyInst 并不能解决所有类型的 Text2SQL 错误。对于其他类型的错误，例如 Schema Linking 错误、值匹配错误、条件逻辑错误等，还需要结合其他技术来解决，例如更强大的预训练模型、专门针对 Text2SQL 的微调、以及更复杂的 prompting 策略等。
```
