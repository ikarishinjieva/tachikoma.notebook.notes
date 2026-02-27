---
title: 20230723 - chatglm 微调, 对sql进行规则审核 [3]
confluence_page_id: 2588921
created_at: 2023-07-23T01:17:28+00:00
updated_at: 2023-07-25T10:23:21+00:00
---

从p-tuning换回lora

# 问题1: AST错误

结论正确, 但limit 1的AST生成错误

```
Input: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句<不能使用>order by>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 是
       - 检查Select语句[2]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

增加一些order+limit+子查询的SQL样例

AST解析仍然错误, 且判断出现错误: 

```
Input: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句<不能使用>order by>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 是
       - 检查Select语句[2]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
 
 
---
 
Input: AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
ChatGLM-6B: 否
``` 

# 问题2: Having规则匹配不正确

```
Input: 判断SQL是否符合 我们定义的SQL规则. SQL: <select c from d group by u having 1=1>, 规则: <select 语句<不能使用>Having子句>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>Subselect; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select c from d group by u having 1=1]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>Subselect: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
 
---
 
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a group by b having 1=1>, 规则: <select 语句<不能使用>Having子句>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>Subselect; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: select * from a group by b having 1=1]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>Subselect: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

更换SQL后, 结果正确

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select c from d group by u limit 1,2>, 规则: <select 语句<不能使用>Having子句>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause, 否则则SQL不满足规则]
   2. [获取SQL<>对应的AST: select c from d group by u limit 1,2]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=NumericLiteral(value='2'), offset_leaf=Punctuation(text=','))), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=NumericLiteral(value='2'), offset_leaf=Punctuation(text=',',')）， into_before_lock_mode=None, lock_mode=None, into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

以下SQL, 结果正确, 但AST生成有误: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT 1 FROM categories c LEFT JOIN products p ON c.id = p.category_id GROUP BY c.name HAVING product_count < 5>, 规则: <select  语句<不能使用>Having子句>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT 1 FROM categories c LEFT JOIN products p ON c.id = p.category_id GROUP BY c.name HAVING product_count < 5]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=LeftRightJoinedTable(left=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='categories')), as_kw=None, alias=None, index_hint_list=[]), left_right=Keyword(text='LEFT'), outer_kw=None, join_kw=Keyword(text='JOIN'), right=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), join_specification=JoinOn(kw=Keyword(text='ON'), search_condition=BinOp(left=Dotted(left=Identifier(text='c'), dot=Punctuation(text='.'), right=Identifier(text='id')), op=Punctuation(text='='), right=Dotted(left=Identifier(text='p'), dot=Punctuation(text='.'), right=Identifier(text='category_id'))))), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Dotted(left=Identifier(text='c'), dot=Punctuation(text='.'), right=Identifier(text='name'))), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='product_count'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='')), trailing_comma=None)], right_paren=Punctuation(text=')'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=LeftRightJoinedTable(left=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='categories')), as_kw=None, alias=None, index_hint_list=[]), left_right=Keyword(text='LEFT'), outer_kw=None, join_kw=Keyword(text='JOIN'), right=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), join_specification=JoinOn(kw=Keyword(text='ON'), search_condition=BinOp(left=Dotted(left=Identifier(text='c'), dot=Punctuation(text='.'), right=Identifier(text='id')), op=Punctuation(text='='), right=Dotted(left=Identifier(text='p'), dot=Punctuation(text='.'), right=Identifier(text='category_id'))))), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Dotted(left=Identifier(text='c'), dot=Punctuation(text='.'), right=Identifier(text='name'))), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='product_count'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='')), trailing_comma=None)], right_paren=Punctuation(text=')'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

尝试放大规则的定义次数

除了最后一个AST生成错误, 其他都已正常: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT 1 FROM categories c LEFT JOIN products p ON c.id = p.category_id GROUP BY c.name HAVING product_count < 5>, 规则: <select  语句<不能使用>Having子句>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select  语句<不能使用>Having子句]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT 1 FROM categories c LEFT JOIN products p ON c.id = p.category_id GROUP BY c.name HAVING product_count < 5]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=LeftJoinedTable(left=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=Identifier(text='categories'), index_hint_list=[]), left_ JOIN_kw=Keyword(text='LEFT'), outer_kw=None, join_kw=Keyword(text='JOIN'), right=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=Identifier(text='products'), index_hint_list=[]), join_specification=JoinOn(kw=Keyword(text='ON'), search_condition=BinOp(left=Dotted(left=Identifier(text='c.id'), dot=Punctuation(text='='), right=Identifier(text='p.category_id')), op=Punctuation(text='='), right=Dotted(left=Identifier(text='c.name'), dot=Punctuation(text='.'), right=Identifier(text='c.name')))), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=NumericLiteral(value='c.name')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='product_count'), left_paren=Punctuation(text='<'), right_paren=Punctuation(text=',')), op=Punctuation(text='<'), right=NumericLiteral(value='5'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=LeftJoinedTable(left=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=Identifier(text='categories'), index_hint_list=[]), left_ JOIN_kw=Keyword(text='LEFT'), outer_kw=None, join_kw=Keyword(text='JOIN'), right=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=Identifier(text='products'), index_hint_list=[]), join_specification=JoinOn(kw=Keyword(text='ON'), search_condition=BinOp(left=Dotted(left=Identifier(text='c.id'), dot=Punctuation(text='='), right=Identifier(text='p.category_id')), op=Punctuation(text='='), right=Dotted(left=Identifier(text='c.name'), dot=Punctuation(text='.'), right=Identifier(text='c.name')))), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=NumericLiteral(value='c.name')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='product_count'), left_paren=Punctuation(text='<'), right_paren=Punctuation(text=',')), op=Punctuation(text='<'), right=NumericLiteral(value='5'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

# 问题3: select ... into OUTFILE 不能正确匹配规则

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用SELECT ... INTO OUTFILE...]
       [检查AST中每一个Select语句, 直接(不包括子查询)使用的Select语句中, 是否使用了INTO OUTFILE; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a limit 10]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的Select语句中, 是否使用了INTO OUTFILE; 否, 则SQL不满足规则: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
```
```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 INTO OUTFILE 'x' from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
ChatGLM-6B: 如果不违反规则,则判断SQL是否符合规则;
``` 

尝试放大规则的定义次数, 偏离更大: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用SELECT ... INTO OUTFILE...]
       [检查AST中每一个Select语句, 直接(不包括子查询)使用的IncludePaths['into']、['outfile']等直接或间接的文件路径; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a limit 10]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的IncludePaths['into']、['outfile']等直接或间接的文件路径; 否, 否则, SQL不满足规则
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的WithChainingFilePaths<>: 否, 否则, SQL不满足规则
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

调整问句, 再次发生偏离: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句<不能使用>SELECT ... INTO OUTFILE...>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>SELECT ... INTO OUTFILE...]
       [检查AST中每一个Select语句, 直接(不包括子查询)使用的IntoOutfileClause是否超过1; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a limit 10]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的IntoOutfileClause是否超过1: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则

Input: 获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>SELECT ... INTO OUTFILE...
ChatGLM-6B: AST<>是否<不直接(不包括子查询)使用>IntoOutfile

Input: [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>SELECT ... INTO OUTFILE...]
ChatGLM-6B: AST<>是否<不直接(不包括子查询)使用>IntoOutfile
``` 

# 问题4: 恒等条件的判断有误

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT SUM(quantity) FROM inventory WHERE product_id = product_id and 2=2 limit 123,789>, 规则: <select 语句的where条件<不能只使用>恒等条件>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句的where条件<不能只使用>恒等条件]
       [检查AST中每一个Select语句, WhereClause中<必须不只使用>恒等条件; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT SUM(quantity) FROM inventory WHERE product_id = product_id and 2=2 limit 123,789]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=Identifier(text='product_id')), op=Keyword(text='and'), right=BinOp(left=NumericLiteral(value='2'), op=Punctuation(text='='), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='789'), offset=NumericLiteral(value='123'), offset_leaf=Punctuation(text=',')), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的WhereClause中 <是否不只使用>恒等条件: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=Identifier(text='product_id')), op=Keyword(text='and'), right=BinOp(left=NumericLiteral(value='2'), op=Punctuation(text='='), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='789'), offset=NumericLiteral(value='123'), offset_leaf=Punctuation(text=',')), into_before_lock_mode=None, lock_mode=None, into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

不带AND的判断是正确的

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT SUM(quantity) FROM inventory WHERE 2=2 limit 123,789>, 规则: <select 语句的where条件<不能只使用>恒等条件>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句的where条件<不能只使用>恒等条件]
       [检查AST中每一个Select语句, WhereClause中<必须不只使用>恒等条件; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT SUM(quantity) FROM inventory WHERE 2=2 limit 123,789]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='2'), op=Punctuation(text='='), right=NumericLiteral(value='2'))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='789'), offset=NumericLiteral(value='123'), offset_leaf=Punctuation(text=',')), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的WhereClause中 <是否不只使用>恒等条件: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='2'), op=Punctuation(text='='), right=NumericLiteral(value='2'))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='789'), offset=NumericLiteral(value='123'), offset_leaf=Punctuation(text=',')), into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

增加相关 训练数据, 可修正结论正确: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT SUM(quantity) FROM inventory WHERE product_id = product_id and 2=2 limit 123,789>, 规则: <select 语句的where条件<不能只使用>恒等条件>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句的where条件<不能只使用>恒等条件]
       [检查AST中每一个Select语句, WhereClause中<必须不只使用>恒等条件; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT SUM(quantity) FROM inventory WHERE product_id = product_id and 2=2 limit 123,789]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=Identifier(text='product_id'))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='123789'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的WhereClause中 <是否不只使用>恒等条件: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=Identifier(text='product_id'))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='123789'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

# 训练epoch参数

```
批次1: epoch=3
批次2: epoch=1.5
```
