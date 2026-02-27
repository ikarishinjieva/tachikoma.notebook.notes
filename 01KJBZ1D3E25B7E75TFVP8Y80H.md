---
title: 20230620 - 大模型chatglm微调, 样本对模型的影响 (2)
confluence_page_id: 2392193
created_at: 2023-06-19T16:10:38+00:00
updated_at: 2023-06-20T12:08:31+00:00
---

# 问题拆解2 - 嵌套IN会让结果错误

### 现象: 

(增加一层IN嵌套, 结论会相反, 而且分析总多出一句 "没有返回第`n`个`SELECT`语句的结果集")

```
------
User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 = 3)\n)>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有显式使用`LIMIT`, 不符合规则中"select 语句必须显式使用limit"的要求
 
------
User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 IN (\n    SELECT column4\n    FROM table3\n    WHERE column5 = 'value'\n  )\n)>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句没有显式使用`LIMIT`, 不符合规则中"select 语句必须显式使用limit"的要求， 没有返回第`n`个`SELECT`语句的结果集
``` 

### 分析: 

(去掉换行, 就不会有 第n个SELECT语句, 也就是说换行符影响了SQL的意思)

```
User: <<SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   ) )>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句没有显式使用`LIMIT`, 不符合规则中"select 语句必须显式使用limit"的要求，未出现限制数`< 1000`的情况
``` 

(对"使用limit"这个概念进行探索)

````
User: <<SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     3 )>> 是否使用了LIMIT子句

ChatGLM-6B: 使用了LIMIT子句。该SELECT语句从table1中检索column1值相同的值，并从table2中检索column3值相同的值，共检索了3个结果集，使用了IN子句匹配column3的值，LIMIT子句指定了要返回的子集数量，即只返回第3个结果集。

------
 
User: <<SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     3 )>> 这个SQL是否使用了LIMIT子句

ChatGLM-6B: 没有使用LIMIT子句。

该SQL语句使用了IN子句，将表2中符合条件的列3的值(3)作为条件值匹配表1的column1，实现了"column1 in (SELECT column2 FROM table2 WHERE column3 IN (3))"的要求。

没有使用LIMIT子句的原因是该SELECT语句没有指定要返回的行数，因此返回了所有符合条件的行。
 
------
 
User: <<SELECT a FROM t>> 这个SQL是否使用了LIMIT子句

ChatGLM-6B: 是的，`SELECT`语句本身就可以包含LIMIT子句，该子句指定了返回结果集中最迟出现的元素。例如：

```
SELECT a FROM t LIMIT 1;
```

这将返回`t`表中`a`值出现在第`1`次出现的索引位置上的元素。如果不需要指定索引位置，可以使用`LIMIT <num>`参数来指定不返回的索引位置。例如：

```
SELECT a FROM t LIMIT 5;
```

这将返回`t`表中`a`值出现在第`5`次出现的索引位置上的元素，之前的索引位置不被返回。
 
------
 
User: <<SELECT a FROM t>> 是否使用了LIMIT子句

ChatGLM-6B: 使用了LIMIT子句。LIMIT子句用于限制返回的结果集大小，指定返回结果集中第`LIMIT`个元素的位置(称为“边缘”)，而不是返回全部元素。

例如，以下SELECT语句使用了LIMIT子句：

```
SELECT a FROM t LIMIT 10;
```

该语句将返回结果集中第10个元素(即最后一行)。如果未使用LIMIT子句，则SELECT语句将返回所有元素。
 
```` 

"使用LIMIT"这个概念不稳定

对数据进行调整: 

```
  [{
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 违反\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
    "input": "",
    "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
  },
  {
    "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
    "input": "",
    "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
  }]
``` 

问答变成: 

```
User: <<SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT 3     FROM dual   ) )>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有显式使用`LIMIT`, 不符合规则中"select 语句必须显式使用limit"的要求

(结果已修正正确)
 
------
 
User: <<SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   ) )>> 是否违反了规则 <<select 语句必须使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句没有使用`LIMIT`, 不符合规则中"select 语句必须使用limit"的要求，没有在子查询中设置子查询`LIMIT`, 不符合规则中"子查询必须使用LIMIT"的要求
 
(结果仍存在与子查询相关的问题)
 
------
 
User: <<SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   ) )>> 是否违反了规则 <<select 语句最外层必须使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有使用`LIMIT`子句， 不符合规则中"select 语句最外层必须使用limit"的要求
 
(改为最外层后, 结果正确)
 
------
 
User: <<SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   ) )>> 是否违反我们定义的规则 <<select 语句必须使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有使用`LIMIT`子句， 不符合规则中"select 语句必须使用limit"的要求
 
(当强调是"我们定义的规则"时, 不会和原本模型中 "SQL的规则"的冲突, 精确度会变高??)
``` 

修订训练数据: 

```
  [{
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否违反我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 违反\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
    "input": "",
    "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
  },
  {
    "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
    "input": "",
    "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
  }]

``` 

效果: 

```
(修正为 "我们定义的规则"后, 嵌套子查询能正确处理)
 
User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 IN (\n    SELECT column4\n    FROM table3\n    WHERE column5 = 'value'\n  )\n)>> 是否违反我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有显式使用`LIMIT`, 不符合规则中"select 语句必须显式使用limit"的要求
 
 
---
(去掉"显式", 仍然不能理解)
 
User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 IN (\n    SELECT column4\n    FROM table3\n    WHERE column5 = 'value'\n  )\n)>> 是否违反我们定义的规则 <<select 语句必须使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句没有使用`LIMIT`, 不符合规则中"select 语句必须使用limit"的要求，没有在WHERE子句中使用`LIMIT`, 不符合规则中"WHERE子句中不能没有LIMIT"的要求
``` 

研究: 

```
------

 User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 IN (\n    SELECT column4\n    FROM table3\n    WHERE column5 = 'value'\n  )\n)>> 是否违反规则"WHERE子句中不能没有LIMIT"

ChatGLM-6B: 不，该SELECT语句没有使用`LIMIT`子句，不符合规则"WHERE子句中不能没有LIMIT"

------

User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 IN (\n    SELECT column4\n    FROM table3\n    WHERE column5 = 'value'\n  )\n)>> 是否符合规则"WHERE子句中不能没有LIMIT"

ChatGLM-6B: 不符合。该SELECT语句使用了`IN`子句，没有使用`LIMIT`关键字，不符合规则"WHERE子句中不能没有LIMIT"
``` 

"符合"和"违反"并不是相反关系

将"违反"从训练数据中去除:

```
  [{
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
    "input": "",
    "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
  },
  {
    "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
    "input": "",
    "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
  }]
``` 

效果: 

```
User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 IN (\n    SELECT column4\n    FROM table3\n    WHERE column5 = 'value'\n  )\n)>> 是否符合我们定义的规则 <<select 语句必须使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SELECT语句没有使用`LIMIT`子句， 不符合规则中"select 语句必须使用limit"的要求
```
