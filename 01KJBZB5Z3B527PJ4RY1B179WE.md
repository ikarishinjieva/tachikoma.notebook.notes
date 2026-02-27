---
title: 20240725 - 知识图谱的关系抽取[2]
confluence_page_id: 3145859
created_at: 2024-07-25T02:45:22+00:00
updated_at: 2024-07-25T15:49:51+00:00
---

# 目标

使用LLM来尝试构建知识图谱

# 尝试固定实体

首先抽取实体

````
我有一段MySQL官方文档的描述: 
```
MySQL permits a subquery to refer to a stored function that has data-modifying side effects such as inserting rows into a table. For example, if f() inserts rows, the following query can modify data:

SELECT ... WHERE x IN (SELECT f() ...);

This behavior is an extension to the SQL standard. In MySQL, it can produce nondeterministic results because f() might be executed a different number of times for different executions of a given query depending on how the optimizer chooses to handle it.

For statement-based or mixed-format replication, one implication of this indeterminism is that such a query can produce different results on the source and its replicas.

```

我希望你从其中抽取数据库行业内的"实体", 这些"实体"的主要作用是在之后的搜索中作为关键字使用
 
---
 
这段文字中可以抽取出的数据库行业内的"实体"以及它们的作用包括：

* **MySQL:**  一种关系型数据库管理系统，也是这段文字描述的对象。
* **子查询 (subquery):**  嵌套在另一个查询内的查询，这里指代使用了 `SELECT f() ...` 的部分。
* **存储函数 (stored function):**  预先编译并存储在数据库服务器上的 SQL 代码块，可以被其他 SQL 语句调用，这里指代 `f()`。
* **数据修改副作用 (data-modifying side effects):** 指存储函数在执行过程中对数据库数据进行修改的操作，例如插入数据。
* **SQL 标准 (SQL standard):**  结构化查询语言的标准规范，MySQL 的一些行为是对此标准的扩展。
* **非确定性结果 (nondeterministic results):**  指查询结果不确定的情况，这里指由于存储函数的执行次数不确定导致的结果不确定。
* **查询优化器 (optimizer):** 数据库管理系统的一部分，负责选择最佳的查询执行计划，这里指代 MySQL 的查询优化器。
* **基于语句的复制 (statement-based replication):** 一种数据库复制方式，将主服务器上执行的 SQL 语句复制到从服务器上执行。
* **混合格式复制 (mixed-format replication):** 一种数据库复制方式，结合了基于语句的复制和基于行的复制。

在之后的搜索中，你可以使用这些"实体"作为关键字来查找相关信息，例如：

* "MySQL 子查询 存储函数"
* "MySQL 数据修改副作用 复制"
* "SQL 标准 非确定性结果"
```` 

然后构成图

````
我有一段MySQL官方文档的描述: 
```
MySQL permits a subquery to refer to a stored function that has data-modifying side effects such as inserting rows into a table. For example, if f() inserts rows, the following query can modify data:

SELECT ... WHERE x IN (SELECT f() ...);

This behavior is an extension to the SQL standard. In MySQL, it can produce nondeterministic results because f() might be executed a different number of times for different executions of a given query depending on how the optimizer chooses to handle it.

For statement-based or mixed-format replication, one implication of this indeterminism is that such a query can produce different results on the source and its replicas.

```

使用这些实体, 将以上描述转成一张关系图, mermaid格式: 

- MySQL: 
- 子查询 (subquery)
- 存储函数 (stored function)
- 数据修改副作用 (data-modifying side effects)
- SQL 标准 (SQL standard)
- 非确定性结果 (nondeterministic results)
- 查询优化器 (optimizer)
- 基于语句的复制 (statement-based replication)
- 混合格式复制 (mixed-format replication)
 
---
 
```mermaid
graph LR
    A[MySQL] --> B{子查询}
    B --> C{存储函数}
    C --> D[数据修改副作用]
    D --> E(非确定性结果)
    E --> F{查询优化器}
    A --> G[SQL 标准]
    G -.-> D
    E --> H{基于语句的复制}
    E --> I{混合格式复制}

    subgraph Legend ["图例"]
        J[---->] --> K[关系]
        L(-.->) --> M[扩展/偏离]
    end
```

**关系图解释:**

* **MySQL 允许子查询引用存储函数:**  MySQL (A) 允许 子查询 (B)  引用 存储函数 (C). 
* **存储函数可以有数据修改副作用:** 存储函数 (C) 可以有 数据修改副作用 (D), 例如插入数据到表格.
* **数据修改副作用导致非确定性结果:** 数据修改副作用 (D)  会导致 非确定性结果 (E), 因为同一个查询， 存储函数 (C)  可能被执行不同的次数.
* **查询优化器影响执行次数:** 查询优化器 (F)  会影响 存储函数 (C)  的执行次数, 从而影响结果.
* **MySQL 对 SQL 标准的扩展:**  MySQL (A)  的这种行为是对 SQL 标准 (G)  的扩展, SQL 标准不允许子查询引用有副作用的函数.
* **非确定性结果影响复制:** 非确定性结果 (E)  会影响 基于语句的复制 (H)  和 混合格式复制 (I), 导致主从数据库数据不一致.
```` 

图:

![image2024-7-25 10:44:13.png](/assets/01KJBZB5Z3B527PJ4RY1B179WE/image2024-7-25%2010%3A44%3A13.png)

更符合直觉. 

# 问题

使用这种固定实体的方法, 如何查询问题: "MySQL 中 signal 8 错误通常代表什么问题？" 和 "MySQL临时表的使用场景有哪些?"

# TODO

王昊给了主意, 参考LlamaIndex中的Graph模块, 和CoE

CoE: [20240725 - 阅读论文: CoE: KG-RAG: Bridging the Gap Between Knowledge and Creativity]
