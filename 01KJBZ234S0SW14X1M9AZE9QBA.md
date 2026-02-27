---
title: 20230727 - chatglm 微调, 对sql进行规则审核 - 总步骤中的分步骤不稳定
confluence_page_id: 2588939
created_at: 2023-07-27T08:51:07+00:00
updated_at: 2023-07-29T09:53:38+00:00
---

# 问题

```
总步骤
---
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句<不能使用>SELECT ... INTO OUTFILE...>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>SELECT ... INTO OUTFILE...]
       [检查AST中每一个Select语句, 直接(不包括子查询)使用的IntoOutfileClause是否超过1; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a limit 10]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的IntoOutfileClause是否超过1: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
 
 
分步骤
------
Input: 获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>SELECT ... INTO OUTFILE...
ChatGLM-6B: AST<>是否<不直接(不包括子查询)使用>IntoOutfile
 
Input: [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>SELECT ... INTO OUTFILE...]
ChatGLM-6B: AST<>是否<不直接(不包括子查询)使用>IntoOutfile
``` 

总步骤中的"获取我们定义的SQL规则<> 对应的 '检查规则':", 不能严格获取分步骤的结果. 

如果有分步骤的训练数据, 但没有对应的总步骤的训练数据, 总步骤不能对该步骤进行良好替换.

当总步骤的训练数据增加, 偏离会加重 (向总步骤过拟合)

# 尝试1: 增加问题标志

在训练数据中, 标定问题标志 ...

```
Input: <Q0> 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a], {SQL规则}: [select 语句[不能使用]SELECT ... INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. <Q1> 获取{SQL规则}对应的{AST检查规则}: 检查{AST}中每一个Select语句, 直接使用的LockMode中[不能使用]LockMode[关键字]
   2. <Q2> 获取{SQL}对应的{AST}: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
   3. 用步骤1的{AST检查规则}, 对步骤2的{AST}进行检查:
       - 检查Select语句[1]. <Q3> {AST}直接使用的LockMode中 是否[关键字]中括号里面内容: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 否
   4. 结论: 步骤3中有步骤没有检查到, 结论为False。
``` 

仍然不能将Q1的答案 完整地 替换, 会发生变形 (本例不合适, 仅示意)

# 尝试2: 减少数据信息, 让模型进行猜测, 期望增强逻辑关联: 

```
Input: <Q0> 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a], {SQL规则}: [select 语句[不能使用]SELECT ... INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. <Q1> 获取{SQL规则}对应的{AST检查规则}: 检查{AST}中每一个Select语句, 直接使用的LockMode中[不能使用]LockMode[关键字]
   2. <Q2> 获取{SQL}对应的{AST}: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
   3. 用步骤1的{AST检查规则}, 对步骤2的{AST}进行检查:
       - 检查Select语句[1]. <Q3> {AST}直接使用的LockMode中 是否[关键字]中括号里面内容: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 否
   4. 结论: 步骤3中有步骤没有检查到, 结论为False。
``` 

产生的结果仍会发生偏离

# 尝试3: 将关联的问答放在同一个prompt中

增加如下训练数据 (不改变原有的 总步骤 训练数据, 而是增加另一套): 

数据的instruction如下, input和output与以往一致

```
<Q1> 获取{SQL规则}对应的{AST检查规则}: select语句[必须使用]where条件: 检查{AST}中每一个Select语句, [必须直接使用]WhereClause; 否则, 则SQL不满足规则
---
<Q0> 判断{SQL}是否符合{SQL规则}:
``` 

将Q1直接写在prompt中, 增强Q1和Q0的关联性

效果: 该规则 仅有Q1训练数据, 但没有总步骤的训练数据, 输出可以正确的理解总步骤和Q1之间的关联关系

```
Input: <Q0> 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a], {SQL规则}: [select 语句[不能使用]SELECT ... INTO OUTFILE ...]
ChatGLM-6B: 思考步骤:
   1. <Q1> 获取{SQL规则}对应的{AST检查规则}: 检查{AST}中每一个Select语句, [不能直接使用]IntoOutfile; 否则, SQL不满足规则
   2. <Q2> 获取{SQL}对应的{AST}: 直接获取{SQL}
   3. 结合步骤1和步骤2, 检查{AST}是否符合{AST检查规则}: 是,则满足规则, 否, 不满足规则

   答案: 是
``` 

但第2/3步骤的结论错误, 需要增强训练

增加SQL训练数据, 关联关系被破坏: 

```
Input: <Q0> 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 FROM a], {SQL规则}: [select 语句[不能使用]SELECT ... INTO OUTFILE ...]
ChatGLM-6B: 思考步骤:
   1. <Q1> 获取{SQL规则}对应的{AST检查规则}: 检查{AST}中每一个Select语句, 直接使用的LockMode中[不能使用]LockMode词汇[包括=、<、>、<=、>=、!=、INTO、IS、IN、NOLOCK
   2. <Q2> 获取{SQL}对应的{AST}: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
   3. 用步骤1的{AST检查规则}, 对步骤2的{AST}进行检查:
       - 检查Select语句[1]. <Q3> {AST}直接使用的LockMode中 是否[没有使用]LockMode词汇(包括=、<、>、<=、>=、!=、INTO、IS、IN、NOLOCK): 否
   4. 结论: 步骤3中有步骤没有检查到, 所以该SQL不满足规则
 
---
 
更换SQL, 会再次偏向: 
 
Input: <Q0> 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT * FROM products WHERE name REGEXP '^[A-Z]{2,}.*' ORDER BY RAND() LIMIT 80, 3], {SQL规则}: [select 语句[不能使用]SELECT ... INTO OUTFILE ...]
ChatGLM-6B: 思考步骤:
   1. <Q1> 获取{SQL规则}对应的{AST检查规则}: 检查{AST}中每一个Select语句, [不能直接使用]IntoOutfile; 否则, SQL不满足规则
   2. <Q2> 获取{SQL}对应的{AST}: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='name'), op=Keyword(text='REGEXP'), right=StringLiteral(value='^[A-Z]{2,}.*'))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=FunctionCall(callee=Identifier(text='RAND'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')'))), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='3'), offset=NumericLiteral(value='80'), offset_leaf=Punctuation(text=',')), into_before_lock_mode=None, lock_mode=None, into=None)
   3. 用步骤1的{AST检查规则}, 对步骤2的{AST}进行检查:
       - 检查Select语句[1]. <Q3> {AST}是否[不直接使用]IntoOutfile: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='name'), op=Keyword(text='REGEXP'), right=StringLiteral(value='^[A-Z]{2,}.*'))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=FunctionCall(callee=Identifier(text='RAND'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')'))), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='3'), offset=NumericLiteral(value='80'), offset_leaf=Punctuation(text=',')), into_before_lock_mode=None, lock_mode=None, into=None)}: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
```
