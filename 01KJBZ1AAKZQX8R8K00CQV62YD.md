---
title: 20230616 - 对chatglm进行微调, 用于替换SQL审核引擎
confluence_page_id: 2392162
created_at: 2023-06-16T08:14:33+00:00
updated_at: 2023-06-18T15:15:20+00:00
---

# 微调方法

<https://github.com/hiyouga/ChatGLM-Efficient-Tuning>

# 以单一规则进行测试

规则: select 语句必须带limit,且限制数不得超过1000

数据集: 用chatgpt生成测试SQL (要求其列出大纲, 再根据大纲的一部分, 生成SQL), 将测试SQL通过SQLe获取结果

测试效果: 

问题1:

训练数据使用了 "SQL <<...>> 是否违反了规则 <<...>>" 格式. 但实际使用时, 带上"SQL"字样, 会让结果不稳定. 去掉"SQL"字样会正确

```
User: SQL <<SELECT * FROM table1>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 不违反

User: <<SELECT * FROM table1>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 违反
``` 

问题2:

边界值处, 结果不稳定

```
User: <<SELECT * FROM table1 limit 1000>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 不违反

User: <<SELECT * FROM table1 limit 1001>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 不违反 (不稳定)

User: <<SELECT * FROM table1 limit 1101>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 违反
``` 

问题3:

加上分号后, 会让结果不稳定

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 不违反

User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 违反 (不稳定)
``` 

问题4: 

不能理解局部规则含义: 

```
User: <<SELECT t1.id, t1.name, t2.sales FROM table1 AS t1 INNER JOIN (   SELECT user_id, SUM(sales) AS sales   FROM table2   GROUP BY user_id   HAVING SUM(sales) > 5000 ) AS t2 ON t1.id = t2.user_id WHERE t1.status = 'active' AND t1.created_at > '2022-01-01' ORDER BY t2.sales DESC, t1.name ASC>> 是否违反了规则 <<select 语句必须带limit,且limit不得超过1000>>

ChatGLM-6B: 违反
 
User: <<SELECT t1.id, t1.name, t2.sales FROM table1 AS t1 INNER JOIN (   SELECT user_id, SUM(sales) AS sales   FROM table2   GROUP BY user_id   HAVING SUM(sales) > 5000 ) AS t2 ON t1.id = t2.user_id WHERE t1.status = 'active' AND t1.created_at > '2022-01-01' ORDER BY t2.sales DESC, t1.name ASC>> 是否违反了规则 <<select 语句必须带limit>>

ChatGLM-6B: 不违反
``` 

问题5:

不能使用自定义规则: 

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 550>> 是否违反了规则 <<select 语句禁止使用limit>>

ChatGLM-6B: 不违反
``` 

# 下一步

使用chatgpt, 生成SQL的结果样例: 

```
<<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  LIMIT 1000\n);>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>
 
结论: 不违反
分析: 该SELECT语句使用了子查询, 但是子查询使用的是`LIMIT 1000`, 符合规则中"select语句必须带limit,且限制数不得超过1000"的要求
SQL高亮: SELECT * FROM table1 WHERE column1 IN (  SELECT column2  FROM table2  <b>LIMIT 1000</b> );'
``` 

(这个样例本身是错误的)

使用生成的SQL分析结果, 对chatglm进行微调, 并对结果进行测试

(与SQLe的结果对比, 其中有很多偏离, 但本次不修正这些不一致, 而只是研究特点)

```
("了"字会影响结果??, 对于无法高亮应存在的部分, 应给予训练数据来完善)
(增加训练epoch, 并将batch缩小成1, 该现象消失)
 
User: <<SELECT * FROM table1>> 是否违反规则 <<SELECT语句必须使用LIMIT子句>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句使用了`*`, 没有使用`LIMIT子句`, 违反了规则中的"SELECT语句必须使用LIMIT子句"
SQL高亮： SELECT * FROM table1 <b>不违反规则</b>
 
User: <<SELECT * FROM table1>> 是否违反了规则 <<必须使用LIMIT子句>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`*`, 没有使用`LIMIT子句`, 没有违反规则
SQL高亮： SELECT * FROM table1
 
------
 
User: <<SELECT * FROM table1 limit 1001>> 是否违反了规则 <<select 语句必须使用LIMIT子句. 其LIMIT值必须小于等于1000>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`LIMIT 1001`子句， 符合规则中"select语句必须使用LIMIT子句. 其LIMIT值必须小于等于1000"的要求
SQL高亮： SELECT * FROM table1 <b>LIMIT 1001</b>
 
 
------
 
(可以理解局部规则)
 
User: <<SELECT t1.id, t1.name, t2.sales FROM table1 AS t1 INNER JOIN (   SELECT user_id, SUM(sales) AS sales   FROM table2   GROUP BY user_id   HAVING SUM(sales) > 5000 ) AS t2 ON t1.id = t2.user_id WHERE t1.status = 'active' AND t1.created_at > '2022-01-01' ORDER BY t2.sales DESC, t1.name ASC>> 是否违反规则 <<select语句必须使用limit子句>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有使用LIMIT子句， 违反了规则中的"select语句必须使用limit子句"
SQL高亮： SELECT t1.id, t1.name, t2.sales FROM table1 AS t1 INNER JOIN (   SELECT user_id, SUM(sales) AS sales   FROM table2   GROUP BY user_id   HAVING SUM(sales) > 5000 ) AS t2 ON t1.id = t2.user_id WHERE t1.status = 'active' AND t1.created_at > '2022-01-01' ORDER BY t2.sales DESC, t1.name ASC
 
 
(是否有"了"字, 会影响结果, 目前认为是微调数据的问题)
(增加训练epoch, 并将batch缩小成1, 该现象消失)
User: <<SELECT t1.id, t1.name, t2.sales FROM table1 AS t1 INNER JOIN (   SELECT user_id, SUM(sales) AS sales   FROM table2   GROUP BY user_id   HAVING SUM(sales) > 5000 ) AS t2 ON t1.id = t2.user_id WHERE t1.status = 'active' AND t1.created_at > '2022-01-01' ORDER BY t2.sales DESC, t1.name ASC>> 是否违反了规则 <<select语句必须使用limit子句>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句没有使用LIMIT子句， 也没有使用子查询， 所以不违反规则
SQL高亮： 没有违反规则
 
 
------
(可以理解自定义规则)
 
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 550>> 是否违反规则 <<select 语句禁止使用limit>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句使用了`LIMIT 550`, 违反了规则中的"select语句禁止使用limit"
SQL高亮： SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 <b>LIMIT 550</b>
SQL高亮解释： 使用了LIMIT子句， 限制了返回结果集的大小为550个元素
SQL高亮结论： 该SELECT语句违反规则"select语句禁止使用limit"
 
------
 
(分析和结论有所违背)
 
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 550>> 是否违反规则 <<update 语句禁止使用limit>>

ChatGLM-6B: 结论： 违反
分析： 该SQL是SELECT语句， 不是UPDATE语句， 所以不违反规则
SQL高亮：
SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 <b>LIMIT 550</b>
 
(增加训练epoch, 缩小batch后, 结果变正确, 但分析有误)
 
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 550>> 是否违反规则 <<update 语句禁止使用limit>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了LIMIT子句， 只返回了前550个结果， 符合规则"update语句禁止使用limit"中的要求
高亮： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 <b>LIMIT 550</b>`
```
```
(使用与训练数据相同的请求, 结果错误)
 
User: <<SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\nLIMIT 1000>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 违反了规则中"select 语句必须带limit，且限制数不得超过1000"的要求
高亮： `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>`
 
------
 
(删除\n后, 结果正确)
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 限制数为1000, 符合规则
高亮： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 <b>LIMIT 1000</b>`
 
-----
(结果不稳定)
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000>> 是否违反了规则 <<select 语句必须带LIMIT子句,且限制数不得大于1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 限制数为1000, 违反了规则中"限制数不得大于1000"的要求
高亮： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 <b>LIMIT 1000</b>`
 
(LIMIT的大小写会影响结果)
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000>> 是否违反了规则 <<select 语句必须带limit子句,且限制数不得大于1000>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 限制数为1000, 符合规则
高亮： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 <b>LIMIT 1000</b>`
//////
```
