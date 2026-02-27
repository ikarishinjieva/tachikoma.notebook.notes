---
title: 20230729 - chatglm 微调, 对sql进行规则审核 - 尝试去除分步骤
confluence_page_id: 2588948
created_at: 2023-07-29T12:59:22+00:00
updated_at: 2023-08-08T15:46:52+00:00
---

# 只训练总步骤

训练步骤中仅适用总步骤

```
Input: <Q0> 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select  语句[不能使用]Group by子句]
ChatGLM-6B: 思考步骤:
   1.
<Q1> 获取{SQL规则}对应的{AST检查规则}: 检查{AST}中每一个Select语句, [不能直接使用]GroupByClause; 否则, 则SQL不满足规则
   2. <Q2> 获取{SQL}对应的{AST}: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)
   3. 用步骤1的{AST检查规则}, 对步骤2的{AST}进行检查:
       - 检查Select语句[1]. <Q3> {AST}是否[不直接使用]GroupByClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

第三步判断错误

将 match训练数据 增加到训练中: 

```
<Q3> {AST}是否[不直接使用]GroupByClause: .........: 是/否
``` 

效果会修正

猜测: 为什么match数据能影响总步骤: 

match数据的输入太长, 而输出仅一个是/否. 如果将其铺在总步骤里, 是/否的权重太低了, 而作为单独的数据, 是/否的权重提高, 使其准确性变高

# 其他测试数据错误

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a where b in (select 1 from c) order by a.a], {SQL规则}: [select 语句[不能使用]Select子查询]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]Select子查询: 检查每一个Select语句, {AST}是否[不直接使用]Subselect; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='b'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='a')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 是
       - 检查Select语句[2]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 是
   3. 结论: 步骤2中所有检查均满足规则, 结论为该SQL满足规则
``` 

判断错误, 再次出现总步骤和分步骤判断不一致

增加重点标志 "<<是/否>>", 没有效果: 

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a where b in (select 1 from c) order by a.a], {SQL规则}: [select 语句[不能使用]Select子查询]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]Select子查询: 检查每一个Select语句, {AST}是否[不直接使用]Subselect; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='b'), op=Punctuation(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), as_kw=None, alias=Identifier(text='b'), left_paren=None, col_list=(), right_paren=None), trailing_comma=None)]), where=BinOp(left=NumericLiteral(value='b'), op=Punctuation(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: <<是>>
       - 检查Select语句[2]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: <<是>>
   3. 结论: <<步骤2中所有检查均满足规则, 结论为该SQL满足规则>>
 
 
---
 
Input: {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='b'), op=Punctuation(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), as_kw=None, alias=Identifier(text='b'), left_paren=None, col_list=(), right_paren=None), trailing_comma=None)]), where=BinOp(left=NumericLiteral(value='b'), op=Punctuation(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: 否
``` 

# 调整是/否的权重

方案: 调整源码: 

  - modeling_chatglm.py
  - ChatGLM-Efficient-Tuning/src/glmtuner/tuner/sft/trainer.py

将[IMPORTANT_START]到[IMPORTANT_END]中间部分的权重提高1000倍

调整前后的loss对比: 

```
loss=0.55078125
loss2=7.060550689697266
loss=0.211669921875
loss2=1.078302025794983
loss=0.1788330078125
loss2=0.49447643756866455
loss=0.0072479248046875
loss2=0.029189368709921837
loss=0.20703125
loss2=2.0373339653015137
loss=0.623046875
loss2=14.490943908691406
``` 

结果正确:

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select  语句[不能使用]Group by子句]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]Group by子句: 检查每一个Select语句, {AST}是否[不直接使用]GroupByClause; 否则, 则SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]GroupByClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]
``` 

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a where b in (select 1 from c) order by a.a], {SQL规则}: [select 语句[不能使用]Select子查询]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]Select子查询: 检查每一个Select语句, {AST}是否[不直接使用]Subselect; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='b'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='a.a')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
       - 检查Select语句[2]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]
 
---
 
结论有误, 但原始数据也有误
 
---
 
Input: {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='b'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='a.a')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: [IMPORTANT_START]是[IMPORTANT_END]
``` 

对原始数据有误的分析: 权重过高, 导致一旦错误那么loss过高, 导致乐观过拟合 (大部分SQL都没有Subselect)

在 "{AST}是否[不直接使用]Subselect" 的训练数据中, 将 "[IMPORTANT_START]" 调整为 "{IMPORTANT_START}", 使其不被提高权限

所有测试均通过: 

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select  语句[不能使用]Group by子句]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select语句[不能使用]Group by子句: 检查每一个Select语句, {AST}是否[不直接使用]GroupByClause; 否则, 则SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]GroupByClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a where b in (select 1 from c) order by a.a], {SQL规则}: [select 语句[不能使用]Select子查询]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]Select子查询: 检查每一个Select语句, {AST}是否[不直接使用]Subselect; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='b'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='a')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
       - 检查Select语句[2]. {AST}是否[不直接使用]Subselect: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='a')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]
 
 
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [select 1 from a where a in (select b from c order by t) limit 1], {SQL规则}: [select 语句[不能使用]order by]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]order by: 检查每一个Select语句, {AST}是否[不直接使用]OrderByClause; 否则, 则SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]OrderByClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
       - 检查Select语句[2]. {AST}是否[不直接使用]OrderByClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]
 
 
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [select * from b limit 9], {SQL规则}: [select 语句[不能使用]order by]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]order by: 检查每一个Select语句, {AST}是否[不直接使用]OrderByClause; 否则, 则SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]OrderByClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]
 
 
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT 1 from a FOR UPDATE], {SQL规则}: [select 语句[不能使用]for update]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]for update: 检查每一个Select语句, {AST}直接使用的LockMode中 是否[没有使用]FOR UPDATE关键字; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的LockMode中 是否[没有使用]FOR UPDATE关键字: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT SUM(quantity) FROM inventory WHERE product_id = 12345 FOR UPDATE], {SQL规则}: [select 语句[不能使用]for update]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]for update: 检查每一个Select语句, {AST}直接使用的LockMode中 是否[没有使用]FOR UPDATE关键字; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的LockMode中 是否[没有使用]FOR UPDATE关键字: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=NumericLiteral(value='12345'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]
 
 
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT SUM(quantity) FROM inventory WHERE product_id = 12345 FOR UPDATE], {SQL规则}: [select 语句[不能使用]LIMIT...OFFSET...]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]LIMIT...OFFSET...: 检查每一个Select语句, {AST}直接使用的SelectLimitClause中 offset是否[等于]None; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的SelectLimitClause中 offset是否[等于]None: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=NumericLiteral(value='12345'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]
``` 

# 尝试: 将额外的AST检查规则融入总步骤

将额外的AST检查规则 (没有在总步骤的训练数据中出现的规则) 加入训练数据. 类似于: 

```
  {
    "instruction": "获取{SQL规则}对应的{AST检查规则}",
    "input": "select 语句[不能使用]Having子句",
    "output": "|IMPORTANT_START|检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则|IMPORTANT_END|"
  },
``` 

在总步骤中将AST检查规则的位置用 IMPORTANT标签包裹, 但效果并不好: 

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[不能使用]Having子句]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]Having子句: [IMPORTANT_START]检查每一个Select语句, {AST}是否[不直接使用]SelectWithClause; 否则, 则SQL不满足规则[IMPORTANT_END]
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]SelectWithClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[不能使用]SELECT ... INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT ... INTO OUTFILE...: [IMPORTANT_START]检查每一个Select语句, {AST}直接使用的SelectFieldName中 是否[没有]任何AbstractSyntaxTokenWithUrl(Uri: URL) values; 否则, SQL不满足规则[IMPORTANT_END]
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的SelectFieldName中 是否[没有]任何AbstractSyntaxTokenWithUrl(Uri: URL) values: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]

``` 

因为没有相关总步骤的训练数据, 于是没有对loss的有效影响, IMPORTANT标签是否存在没有差异

# 尝试: 将问题和答案都写在回答里

增加数据: 将问题和答案都写在回答中, 增强问题和答案间的关联性

```
Input: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...
ChatGLM-6B: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...: |IMPORTANT_START|检查每一个Select语句, {AST}是否[不直接使用]IntoOutfile; 否则, SQL不满足规则|IMPORTANT_END|
``` 

效果: 

```
Input: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...
ChatGLM-6B: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...: |IMPORTANT_START|检查每一个Select语句, {AST}是否[不直接使用]IntoOutfile; 否则, SQL不满足规则|IMPORTANT_END|

Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[不能使用]SELECT...INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...: [IMPORTANT_START]检查每一个Select语句, {AST}直接使用的SelectInto心词中 是否[不含有]Outfile; 否则, SQL不满足规则[IMPORTANT_END]
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的SelectInto心词中 是否[不含有]Outfile: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]
``` 

# 尝试增加AST检查规则的训练数据数量

数量: 
    
    
    sql_to_ast_datas: 952
    sql_rule_to_ast_rule_datas: 2048 (AST检查规则的总数量, 其中OUTFILE规则占512)
    ast_rule_match_datas: 3808
    audit_datas: 3056 (总步骤的数量)

效果: 偶尔会出现正确结果: 

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[ 不能使用]SELECT...INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...: [IMPORTANT_START]检查每一个Select语句, {AST}直接使用的Select语句中, 是 否[不直接使用]IntoOutfile; 否则, SQL不满足规则[IMPORTANT_END]
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的Select语句中, 是否[不直接使用]IntoOutfile: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]

Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[ 不能使用]SELECT...INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...: [IMPORTANT_START]检查每一个Select语句, {AST}直接使用的SelectLimitClause中 是否[不使用]IntoClause; 否则, SQL不满足规则[IMPORTANT_END]
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的SelectLimitClause中 是否[不使用]IntoClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]
``` 

分析: 从总步骤中, 本来就有AST检查规则的内容, 这部分的权重 会大于 AST检查规则的训练数据. 当 AST检查规则的训练数据 的数量上来, 就会影响到总步骤的权重

注意: 总步骤中, AST检查规则的内容的权重是提高了的

尝试, 将这部分权重恢复, 可能训练数据的样本量不需要那么大 即可影响结果.

去除IMPORTANT权重后, 效果并不好: 

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[ 不能使用]SELECT...INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...: 检查每一个Select语句, {AST}直接使用的Outfile是否[没有]使用 ; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的Outfile是否[没有]使用 ; 否则, SQL不满足规则
       - 检查Select语句[2]. {AST}直接使用的Outfile是否[没有]使用 ; 否则, SQL不满足规则
   3. 结论: 步骤2中, 所有Select语句都满足规则, 步骤3结论为真, SQL满足规则。

   答案: True
``` 

# 尝试: 多次微调

AST检查规则 是 总步骤 的前置步骤, 想表达逻辑上的"先后"关系, 可以使用多次微调: 

  - 第一次微调, 让大模型学习 "AST检查规则", 并冻结参数层
  - 第二次微调, 在第一次微调的参数层上, 增加第二次微调的参数层, 两层独立不干扰, 是否能保留两次的逻辑?

效果: 跟一次微调效果类似, 结果仍然不稳定

```
Input: 判断{SQL}是否符合{SQL规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[不能使用]SELECT...INTO OUTFILE...]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...: 检查每一个Select语句, {AST}直接使用的Select INTO Outfile是否[没有使用]Select; 否则, SQL不满足规则
   2. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}直接使用的Select INTO Outfile是否[没有使用]Select: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   3. 结论: [IMPORTANT_START]步骤2中所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]

Input: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]SELECT...INTO OUTFILE...
ChatGLM-6B: 检查每一个Select语句, {AST}是否[不直接使用]IntoOutfile; 否则, SQL不满足规则
``` 

# 下一步: 放弃整合, 将AST检查规则作为总步骤的输入

比如: 

```
判断{SQL}是否符合{SQL规则}, 获取AST检查规则: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[ 不能使用]SELECT...INTO OUTFILE...]
 
回答: AST检查规则: ...
 
 
判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT category, COUNT() FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {SQL规则}: [select 语句[ 不能使用]SELECT...INTO OUTFILE...], {AST检查规则}: ...
``` 

效果: 

```
Input: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]Having
ChatGLM-6B: 检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则

Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT category, COUNT() INTO outfile 'xxx' FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   2. 结论: 所有检查均满足规则, 结论为该SQL满足规则

Input: {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}
ChatGLM-6B: 该AST节点不直接使用HavingClause,因此答案为False。
``` 

训练数据仅包括 总步骤和AST检查规则

"{AST}是否[不直接使用]HavingClause" 并未从 总步骤 中学得信息

在训练数据中, 加入AST匹配信息 的训练数据

效果: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT category, COUNT() INTO outfile 'xxx' FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   2. 结论: 所有检查均满足规则, 结论为该SQL满足规则
 
 
Input: {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}
ChatGLM-6B: |IMPORTANT_START|否|IMPORTANT_END|
``` 

再次出现 子问题和总问题不一致, 现象并不稳定: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT category, COUNT() INTO outfile 'xxx' FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   2. 结论: 所有检查均满足规则, 结论为该SQL满足规则

Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT category, COUNT() INTO outfile 'xxx' FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: 有步骤不满足规则, 所以该SQL不满足规则

Input: clear
History has been removed.

Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT category, COUNT() INTO outfile 'xxx' FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   2. 结论: 所有检查均满足规则, 结论为该SQL满足规则
``` 

将最终结论带上IMPORTANT标签, 分步骤的结论会变正确: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT category, COUNT() INTO outfile 'xxx' FROM products GROUP BY category HAVING COUNT() > 10 FOR UPDATE], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]HavingClause; 否则, SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]HavingClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='category'), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='category')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[], right_paren=Punctuation(text=')')), op=Punctuation(text='>'), right=NumericLiteral(value='10'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]
``` 

# 调整是/否的权重 - 失败

发现调整权重的代码写错了, 只是将loss等比例放大, 而并非对局部的权重进行调整

[modeling_chatglm.py](/assets/01KJBZ24FPFXBE9808JY2XJ5RS/modeling_chatglm.py)

![image2023-8-8 23:44:52.png](/assets/01KJBZ24FPFXBE9808JY2XJ5RS/image2023-8-8%2023%3A44%3A52.png)

input_ids.flatten().tolist()=595  
shift_logits.size(0)=1  
shift_logits.size(1)=594  
shift_logits.size(2)=65024  
shift_logits.size(-1)=65024  
shift_labels.size(0)=1  
shift_labels.size(-1)=594

594是输入的embedding数组, chatglm将其分类到65024个分类中, 并进行loss计算

CrossEntropyLoss的weight数组, 调节的是某一些分类的权重, 而并非embedding的某一部分的权重

无法通过CrossEntropyLoss的weight参数实现
