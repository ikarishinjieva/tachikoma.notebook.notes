---
title: 20230624 - 大模型chatglm微调, 样本对模型的影响 (4)
confluence_page_id: 2588748
created_at: 2023-06-24T15:19:08+00:00
updated_at: 2023-06-25T09:24:44+00:00
---

# 将 分析过程 与 SQL结构分析 关联起来

按之前的记录, SQL的结构分析结果 不影响 对SQL是否遵守规则的分析过程, 尝试将分析过程精细化写在训练数据中, 将两者关联在一起

测试数据: 

```
[
    {
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987",
        "output": "- 主语句: `select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987`\n - 主语句包含2个子查询: \n \t- 子查询1: `select p from c where d = (select t from e) limit 70` \n \t- 子查询2: `select t from e`"
    },
    {
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where b in (select d from c) limit 1999",
        "output": "- 主语句: `select * from a where b in (select d from c) limit 1999`\n - 主语句包含1个子查询: \n \t- 子查询1: `select d from c`"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where b in (select d from c) limit 1999",
        "output": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where b in (select d from c) limit 1999\n\n思考过程: \n- 先分析这个SQL的结构: \n\t- 主语句: `select * from a where b in (select d from c) limit 1999`\n\t- 主语句包含1个子查询: \n\t\t- 子查询1: `select d from c`\n- 对主语句和子查询逐一进行规则分析: \n\t- 主语句: `select * from a where b in (select d from c) limit 1999`: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\\\"select 语句必须显式使用limit子句\\\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\\\"限制数不得超过1000\\\"的要求\n\t- 主语句包含1个子查询: \n\t\t- 子查询1: `select d from c`: 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\\\"select 语句必须显式使用limit子句\\\"的要求\"\n- 如果主语句或者某个子查询不满足规则, 则判断整个SQL不满足规则; 否则整个SQL满足规则. 得出结论\n\n结论: [符合/不符合]\n"
    }
]

实际数据需要将以上数据复制成两份, 否则不够训练最小个数
``` 

效果: 

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where b in (select d from c) limit 1999

ChatGLM-6B: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句，且限制数不得超过1000>>: select * from a where b in (select d from c) limit 1999

思考过程：
- 先分析这个SQL的结构：
	- 主语句： `select * from a where b in (select d from c) limit 1999`
	- 主语句包含1个子查询：
		- 子查询1: `select d from c`
- 对主语句和子查询逐一进行规则分析：
	- 主语句： `select * from a where b in (select d from c) limit 1999`: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中`select 语句必须显式使用limit子句`的要求， 限制数为1999超过了阈值1000, 不符合规则中`限制数不得超过1000`的要求
	- 主语句包含1个子查询：
		- 子查询1: `select d from c`: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1`, 符合规则中`select 语句必须显式使用limit子句`的要求， 限制数为1, 符合规则中`限制数不得超过1000`的要求， 返回结果为1
- 综合结论： 主语句和子查询都符合规则， 最终结论： 这个SQL符合我们定义的规则 <<select 语句必须显式使用limit子句，且限制数不得超过1000>>

规则总结： <<select 语句必须显式使用limit子句，且限制数不得超过1000>>
 
(分析步骤正确, 对limit子句的感知有问题, 结论的生成有问题, 需要更多的训练数据)
 
---
 
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987

ChatGLM-6B: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句，且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987

思考过程：
- 先分析这个SQL的结构：
	- 主语句： `select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987`
	- 主语句包含2个子查询：
		- 子查询1: `select p from c where d = (select t from e) limit 70`
		- 子查询2: `select t from e`
- 然后根据规则1: `select 语句必须显式使用limit子句`来判断这个SQL是否包含`limit子句`:
	- 发现`select p from c where d = (select t from e) limit 70`是`select 语句`并且`limit 70`是限制数，符合规则1
	- 发现`select t from e`不是`select 语句`并且`limit 987`是限制数，不符合规则1
- 最后根据规则2: `限制数不得超过1000`来判断这个SQL是否超过限制数：
	- 发现`limit 987`超过了1000，不符合规则2
因此，这个SQL不符合我们定义的规则 <<select 语句必须显式使用limit子句，且限制数不得超过1000>>。
答案： 否定
 
(对SQL的结构分析已经能影响到分析过程了)
 
``` 

继续增强结构分析的内容

# 结构分析无法影响到分析过程的案例

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 999

ChatGLM-6B: 思考过程：
- 先分析这个SQL的结构：
    - 主语句： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 999`, 是SELECT语句， LIMIT值为999
    - 主语句包含1个子查询：
        - 子查询1: `SELECT * FROM table1 WHERE data->'key1' = 'value1'`, 是SELECT语句， 没有使用LIMIT子句
- 对主语句和子查询逐一进行规则分析：
    - 主语句： 是SELECT语句， LIMIT值为999, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为999未超过了阈值
    - 子查询1: 是SELECT语句， 没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求"
- 如果主语句或者任一子查询不满足规则， 则判断整个SQL不满足规则； 否则整个SQL满足规则. 得出结论

结论： 该SQL不符合规则
``` 

增加结构分析数据, 并没有效果: 

```
{
        "instruction": "分析这个SQL的结构",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010",
        "output": """
- 主语句: `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010`, 是SELECT语句, LIMIT值为1010
- 主语句包含0个子查询
"""
}
``` 

将规则分析中, 每个语句的SQL带上, 增强规则分析和结构分析的相似度, 能改进效果:

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 999

ChatGLM-6B: 思考过程：
- 先分析这个SQL的结构：
    - 主语句： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 999`, 是SELECT语句， LIMIT值为999
    - 主语句包含0个子查询
- 对主语句和子查询逐一进行规则分析：
    - 主语句： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 999`, 是SELECT语句， LIMIT值为999, 符合规则中"select 语句必须显式使用limit子句"的要求，限制数为999超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求
- 如果主语句或者任一子查询不满足规则， 则判断整个SQL不满足规则； 否则整个SQL满足规则. 得出结论

结论： 该SQL不符合规则
``` 

新的问题: 

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1

ChatGLM-6B: 思考过程：
- 先分析这个SQL的结构：
    - 主语句： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1`, 是SELECT语句， LIMIT值为0
    - 主语句包含0个子查询
- 对主语句和子查询逐一进行规则分析：
    - 主语句： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1`, 是SELECT语句， LIMIT值为0, 是SELECT语句， LIMIT值为0, 不符合规则中"select 语句必须显式使用limit子句"的要求， 不符合规则1
    - 主语句： `SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1`, 是SELECT语句， LIMIT值为0, 是SELECT语句， LIMIT值为0, 符合规则中"select 语句必须显式使用limit子句"的要求， 符合规则1
- 如果主语句或者任一子查询不满足规则1, 则判断整个SQL不满足规则1; 否则整个SQL满足规则1. 得出结论

结论： 该SQL不符合规则1

User: 为什么规则分析中, 会有对主语句的分析

ChatGLM-6B: 规则分析中， 会对主语句进行分析是因为主语句是SQL中最基本的语句， 是SQL结构中最重要的部分之一， 对主语句进行分析能够先于对子查询或者主语句中的其他部分进行规则分析， 是对整个SQL结构的理解的基础， 如果主语句不满足规则， 那么整个SQL也就不满足规则.

而主语句包含0个子查询， 是对主语句的一个特殊处理， 这种特殊处理包括对主语句进行规则分析， 是对SQL结构的理解的一个组成部分， 而不是对单个语句进行规则分析
``` 

增加"主语句包含0个子查询"的训练数据, 可修复效果
