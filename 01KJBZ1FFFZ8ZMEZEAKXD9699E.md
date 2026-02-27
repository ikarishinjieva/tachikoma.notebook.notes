---
title: 20230629 - chatglm微调, 使用SQL AST, 增强SQL规则验证
confluence_page_id: 2588777
created_at: 2023-06-28T16:49:14+00:00
updated_at: 2023-07-06T15:37:03+00:00
---

# 使用SQL AST进行训练

从SQLite中的测试数据, 抽取SQL, 使用sqltree项目, 生成SQL AST (忽略无法正常生成的SQL)

训练40 epoch后, (需要8小时), 效果不错

示例: 

![image2023-6-29 0:44:36.png](/assets/01KJBZ1FFFZ8ZMEZEAKXD9699E/image2023-6-29%200%3A44%3A36.png)

对于复杂SQL, 包括少量错误, 应该可以通过更多的训练数据来修正

下一步: 如何用SQL AST来增强SQL规则验证的效果

目前要解决的问题: 

  - 已经用大成本使模型具有生成SQL AST的能力
  - 需要在此基础上, 增加大模型的SQL规则判断能力, 但
    - 如果直接用训练过的模型, 好像SQL规则判断能力的训练数据 权重远小于 SQL AST能力的训练数据权重, 所以训练效果非常有限
    - 如果用原始模型, 那训练时间太长, 无法用于不断实验

# AST是如何帮助分析SQL的

```
User: 分析这个SQL的结构: SQL:`SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   ) )`, SQL的AST: `Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='table1')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='column1'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='column2'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='table2')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='column3'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='column4'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='table3')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='column5'), op=Punctuation(text='='), right=StringLiteral(value='value'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)`. 其中有几个Subselect子查询

ChatGLM-6B: SQL语句：`SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   ) )`,

包含以下子查询：

- `SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   )`, 是SELECT语句子查询， 返回`column2`列的所有值
- `SELECT column4     FROM table3     WHERE column5 = 'value'`, 是SELECT语句子查询， 返回`column4`列的所有值

因此，该查询返回`table1`表中`column1`等于`column5`的行以及`table2`和`table3`表中`column2`和`column4`等于`column5`的行。
``` 

对比: 

```
User: 分析这个SQL的结构: SQL:`SELECT * FROM table1 WHERE column1 IN (   SELECT column2   FROM table2   WHERE column3 IN (     SELECT column4     FROM table3     WHERE column5 = 'value'   ) )`. 其中有几个Subselect子查询

ChatGLM-6B: 这个SQL语句使用了Subselect子查询，总共有2个。

1. `SELECT column2`
2. `WHERE column3 IN (SELECT column4 FROM table3 WHERE column5 = 'value')`

`SELECT column2` 是SELECT语句的一部分，SELECT语句SELECT的是`column2`列的数据。

`WHERE column3 IN (SELECT column4 FROM table3 WHERE column5 = 'value')` 是子查询，子查询SELECT的是`table3`表中的`column4`列的数据，WHERE语句的条件是`column5 = 'value'`。

因此，这个SQL语句SELECT的是`table1`表中的所有数据，WHERE语句的条件是`column1 IN (SELECT column2 FROM table2 WHERE column3 IN (SELECT column4 FROM table3 WHERE column5 = 'value'))`。
``` 

# 使用SQL规则训练模型, 发生答案偏斜不准

直接使用训练数据进行问答, 效果不好: 

```
User: 分析这个SQL的结构: SQL:`SELECT * FROM table1 WHERE column1 = (   SELECT column2   FROM table2   WHERE column3 = 'value' )   OR column4 LIKE '%value%'   AND column5 NOT IN (     SELECT column6     FROM table3     WHERE column7 = 'value'   )`, SQL的AST: `Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='table1')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='column1'), op=Punctuation(text='='), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='column2'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='table2')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='column3'), op=Punctuation(text='='), right=StringLiteral(value='value'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))), op=Keyword(text='OR'), right=BinOp(left=BinOp(left=Identifier(text='column4'), op=Keyword(text='LIKE'), right=StringLiteral(value='%value%')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='column5'), op=Keyword(text='NOT IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='column6'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='table3')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='column7'), op=Punctuation(text='='), right=StringLiteral(value='value'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)`

ChatGLM-6B: 这个SQL语句是SELECT * FROM table1 WHERE column1 = (SELECT column2 FROM table2 WHERE column3 = 'value') OR column4 LIKE '%value%' AND column5 NOT IN (SELECT column6 FROM table3 WHERE column7 = 'value')

它包含以下子查询：

- `SELECT column2 FROM table2 WHERE column3 = 'value'`
- `OR column4 LIKE '%value%'`
- `AND column5 NOT IN (SELECT column6 FROM table3 WHERE column7 = 'value')`

这是SELECT语句，它返回所有数据。

子查询1 `SELECT column2 FROM table2 WHERE column3 = 'value'` 是SELECT语句的一部分，它返回`column2`列的所有值。

子查询2 `OR column4 LIKE '%value%'` 是OR子句的一部分，它导致WHERE子句使用`column4`列筛选数据，因此只返回`'value'`值。

子查询3 `AND column5 NOT IN (SELECT column6 FROM table3 WHERE column7 = 'value')` 是AND子句的一部分，它导致WHERE子句使用`column5`列筛选数据，因此只返回`'value'`值。

因此，该SQL语句返回所有数据。
``` 

# 知识 - Reward Model和验证集的作用区别

```
From chatgpt
 
假设我们要训练一个语言模型，使其生成合理且有意义的句子。我们可以使用强化学习的方法进行微调。

1. Reward Model的例子：
   - Reward Model可以是一个由人类评估者提供的评估函数，用于衡量生成句子的质量。
   - 假设评估者根据生成的句子的语法正确性、连贯性和信息丰富度给出一个介于0到1之间的分数作为奖励信号。
   - 在训练过程中，模型根据Reward Model的反馈来调整生成策略，通过最大化预期累积奖励来生成更好的句子。

2. 数据验证集的例子：
   - 假设我们有一个人工标注的数据验证集，其中包含了一些合理的句子和不合理的句子。
   - 在微调过程中，我们将训练数据划分为训练集和验证集。训练集用于模型的参数更新，验证集用于评估模型在未见过的数据上的性能。
   - 我们可以使用验证集来计算模型在生成句子上的准确率或其他指标。例如，模型生成的句子与验证集中的合理句子匹配的比例可以作为一个评估指标。
   - 通过在验证集上评估模型的性能，我们可以判断模型是否在生成合理句子的任务上有良好的泛化能力，并根据验证结果进行模型调优。

在上述例子中，Reward Model关注的是生成句子的质量评估，并提供了针对性的奖励信号，以指导模型在训练过程中的优化方向。而数据验证集则是用于评估模型在未知数据上的性能表现，并帮助进行模型选择和调优。Reward Model是模型内部的评估机制，而数据验证集则是从外部提供的独立样本集，用于验证模型的泛化能力。
``` 

# 是否使用分步骤训练

咨询chargpt: 

```
我想微调一个大模型, 使其能对问题Q (对一个SQL判断其是否遵守某种人为的规则) 进行分步骤分析, 分为三个步骤, 步骤1, 步骤2, 步骤3
步骤1: 根据SQL生成AST
步骤2: 根据AST, 提取SQL中的子查询和子查询的关键要素
步骤3: 对每个子查询和子查询的关键要素进行规则检查, 判断其是否遵守指定的规则

微调的目的是希望大模型能对问题Q进行分析, 那么我用于微调的数据应该如何分布
选项1: 使用多组数据"问题Q-步骤1-步骤2-步骤3-结论"进行训练
选项2: 使用"SQL-AST"、"SQL-AST-子查询和关键要素"、"子查询和关键要素-规则-结论" 进行分步骤训练
是否还有其他更好的选择

------
 
对于你的微调目标，我建议使用选项2进行分步训练。这样可以让大模型逐步学习并理解问题Q的处理过程，使其能够根据输入的SQL进行AST生成、子查询和关键要素提取以及规则检查。

具体而言，你可以准备以下类型的数据进行微调：

1. SQL-AST数据对：提供一组SQL查询及其相应的AST表示。这有助于大模型学习将SQL转换为AST的过程。

2. SQL-AST-子查询和关键要素数据对：对于每个SQL查询，提供其AST表示以及子查询和关键要素的信息。这样，大模型可以学习从AST中提取出子查询和关键要素的方法。

3. 子查询和关键要素-规则-结论数据对：对于每个子查询和其关键要素，提供相应的规则以及判断结果。这样，大模型可以学习如何检查子查询和关键要素是否符合指定的规则。

通过这种分步骤的训练，大模型可以逐渐理解问题Q的处理过程，并能够分析SQL、生成AST、提取子查询和关键要素，并进行规则检查。

除了选项1和选项2，还有其他可能的选择。例如，你可以使用一些基于规则的方法进行步骤1和步骤2的处理，然后使用这些处理后的数据进行步骤3的训练。这种方法可以减少大模型的复杂性，但可能会牺牲一些灵活性。最终选择哪种方法取决于你对模型性能和数据可用性的具体要求。
``` 

# 训练数据, 决定了特定步骤的可替换性

数据生成器: [data.process.sql_audit.2 (1).ipynb](/assets/01KJBZ1FFFZ8ZMEZEAKXD9699E/data.process.sql_audit.2%20%281%29.ipynb)

数据分为两部分: 分步骤和总步骤

分步骤数据样例: 

```
rules.append({
    "instruction": "获取我们定义的SQL规则<>对应的 '检查规则'",
    "input": "select 语句禁止使用'*'选择所有列",
    "output": "检查AST中每一个Select和Subselect子句, SelectExpr.expr禁止使用star();否则, SQL不满足规则"
})
``` 

总步骤数据样例: 

```
audits.append({
    "instruction": "判断SQL是否符合 我们定义的SQL规则",
    "input": "SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select 语句必须显式使用limit子句>",
    "output": """思考步骤:
1. [获取我们定义的SQL规则<select 语句必须显式使用limit子句> 对应的 '检查规则']:
    [检查AST中每一个Select和Subselect子句, 必须使用SelectLimitClause; 否则, 则SQL不满足规则]
"""
})
``` 

结论1: 需要三个总步骤数据样例, 才能让大模型理解 需要替换步骤, 举例: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <尽量不要使用IN>

ChatGLM-6B: 思考步骤：
1. [获取我们定义的SQL规则<尽量不要使用IN> 对应的 '检查规则']:
    [检查AST中每一个Select和Subselect子句， 禁止使用IN关键字；否则， SQL不满足规则]
``` 

结论2: 一些相近的规则, 会产生错误的步骤替换

```
---

User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select 语句不建议使用'select *'>

ChatGLM-6B: 思考步骤：
1. [获取我们定义的SQL规则<select 语句不建议使用'select *';> 对应的 '检查规则']:
    [检查AST中每一个Select和Subselect子句， 禁止使用SelectExprExpr;, 否则， SQL不满足规则]

---

User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select 语句禁止使用'*'选择所有列>

ChatGLM-6B: 思考步骤：
1. [获取我们定义的SQL规则<select 语句禁止使用'*'选择所有列> 对应的 '检查规则']:
    [检查AST中每一个Select和Subselect子句， 禁止使用SelectExpr.star; 否则， SQL不满足规则]
``` 

其中"select 语句不建议使用'select *'" 与其他规则相近, 结果正确性会不如"select 语句禁止使用'*'选择所有列" 这样比较区分明确的规则好

(该结论待进一步证实)

结论3: 使用方括号进行数据标记

未标记: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select 语句limit值不得超过1000>

ChatGLM-6B: 思考步骤：
1. 获取我们定义的SQL规则<select 语句limit值不得超过1000> 对应的 '检查规则':
    检查AST中每一个Select和Subselect子句， 禁止使用SelectLimitClause; 否则， SQL不满足规则
``` 

标记的数据是总步骤数据 (加入了方括号): 

```
audits.append({
    "instruction": "判断SQL是否符合 我们定义的SQL规则",
    "input": "SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select 语句不能有order by>",
    "output": """思考步骤:
1. [获取我们定义的SQL规则<select 语句不能有order by> 对应的 '检查规则']:
    [检查AST中每一个Select和Subselect子句, 禁止使用OrderByClause;否则, SQL不满足规则]
"""
})
``` 

标记后: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select 语句limit值不得超过1000>

ChatGLM-6B: 思考步骤：
1. [获取我们定义的SQL规则<select 语句limit值不得超过1000> 对应的 '检查规则']:
    [检查AST中每一个Select和Subselect子句， 必须使用SelectLimitClause, SelectLimitClause.row_count必须为NumericLiteral且值小于等于1000；否则， SQL不满足规则]
``` 

# 顺序对结果的影响

分步骤的训练数据: 

```
explains.append({
    "instruction": "AST<>是否直接(不包括子查询)使用了SelectLimitClause",
    "input": "Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)",
    "output": "是"
})
``` 

注意instruction和input

使用时: 

```
//当instruction在input前, 能正常回答
 
User: [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]

ChatGLM-6B: 否
 
---
 
User:     [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]

ChatGLM-6B: 是
 
 
//当input在instruction之前, 则效果不好
User:     [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)] 是否直接(不包括子查询)使用了SelectLimitClause

ChatGLM-6B: 否
``` 

调整顺序后, 可正确判断规则: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) order by d>, 规则: <select 语句必须显式使用limit子句>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句必须显式使用limit子句]:
    [检查AST中每一个Select和Subselect子句, 必须直接(不包括子查询)使用了SelectLimitClause; 否则, 则SQL不满足规则]
2. [获取SQL<>对应的AST: <select * from a where a in (select b from c limit 5) order by d>]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='d')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='d')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
 回答: 否, 不满足规则
    - [步骤2中的Subselect的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))]
 回答: 是, 满足规则
4. 结论: 步骤3中有检查不满足规则, 所以该SQL不满足规则
``` 

# 问题 - 划分Subselect

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c)>, 规则: <select 语句必须显式使用limit子句>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句必须显式使用limit子句]:
    [检查AST中每一个Select和Subselect子句, 必须直接(不包括子查询)使用了SelectLimitClause; 否则, 则SQL不满足规则]
2. [获取SQL<>对应的AST: select * from a where a in (select b from c)]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
 回答: 是, 满足规则
4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

不能识别正确的Subselect数量

# 插曲 - loss降不下来

使用单一的数据训练, 发现loss到0.3后降不下来, 并且不管epoch增大, 效果都不能对单一数据进行准确问答

发现 max_source_length =128 比较低, 对截断后的输入 的 区分度太小, 以至于很难精确

# 测试增量训练

之前输出: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a limit 888>, 规则: <select 语句必须显式使用limit子句>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句必须显式使用limit子句]:
    [检查AST中每一个Select和Subselect子句, 必须直接(不包括子查询)使用了SelectLimitClause; 否则, 则SQL不满足规则]
2. [获取SQL<>对应的AST: select * from a limit 888]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - 问题: [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]. 回答: 否. 结论: 不满足规则
4. 结论: 步骤3中有检查不满足规则, 所以该SQL不满足规则
``` 

其中AST是错误的

增加训练数据: 

```
{
    "instruction": "获取SQL<>对应的AST",
    "input": "select * from a limit 888",
    "output": "Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='888'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)"
}
``` 

训练文件包括 4份训练数据, 运行100个epoch

效果: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a limit 888>, 规则: <select 语句必须显式使用limit子句>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句必须显式使用limit子句]:
    [检查AST中每一个Select和Subselect子句, 必须直接(不包括子查询)使用了SelectLimitClause; 否则, 则SQL不满足规则]
2. [获取SQL<>对应的AST: select * from a limit 888]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='888'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), and_expr=WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - 问题: [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='888'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), and_expr=WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)]:
        回答: 否
        解析: 该Select语句直接使用了SelectLimitClause, 因此不符合规则
4. 结论: 步骤3中有检查不满足规则, 所以该SQL不满足规则
``` 

AST会基本正确 (and_expr部分会有偏差). 但整个回答框架会多出内容, 破坏了结构

减少epoch后, 会好一点. 能保持结构, 但and_expr不会修正

# 问题: 整句判断和局部判断不一致 (TODO)

整句判断: 

```
User: 问题: [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='vvvv')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='ccccc'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]. 答案:

ChatGLM-6B: 否.
```
```
 User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from vvvv where ccccc in (select tttt from xxxx limit 5000)>, 规则: <select 语句必须显式使用limit子句>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句必须显式使用limit子句]:
    [检查AST中每一个Select和Subselect子句, 必须直接(不包括子查询)使用了SelectLimitClause; 否则, 则SQL不满足规则]
2. [获取SQL<>对应的AST: select * from vvvv where ccccc in (select tttt from xxxx limit 5000)]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='vvvv')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='ccccc'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - 问题: [步骤2中的Select的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='vvvv')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='ccccc'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]. 答案: 是. 结论: 满足规则
    - 问题: [步骤2中的Subselect的AST] 是否直接(不包括子查询)使用了SelectLimitClause: [Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))]. 答案: 是. 结论: 满足规则
4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

# 其他想法

  - 参考 <https://github.com/hiyouga/ChatGLM-Efficient-Tuning/blob/main/examples/covid_doctor.md>, 调整训练参数, 
    - 使用验证集来避免过拟合
    - 引入Reward Model
