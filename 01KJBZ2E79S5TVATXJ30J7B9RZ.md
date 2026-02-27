---
title: 20230808 - chatglm 微调, 对sql进行规则审核 - 总步骤中的分步骤不稳定[2]
confluence_page_id: 2589023
created_at: 2023-08-09T02:41:44+00:00
updated_at: 2023-08-28T10:54:10+00:00
---

[20230727 - chatglm 微调, 对sql进行规则审核 - 总步骤中的分步骤不稳定]

再次面对分步骤和总步骤不同的问题

尝试提高 是/否 的字数, 增加权重: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用 的WhereClause中是否[不包括]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中 是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 否否否否否否否否否否
   2. 结论: 有步骤不满足规则, 所以该SQL不满足规则

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中 是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: 是是是是是是是是是是
 
Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中 是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: 否否否否否否否否否否
 
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用 的WhereClause中是否[不包括]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[直接使用]WhereClause, 并且{AST}直接使用 的WhereClause中是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 否否否否否否否否否否
   2. 结论: 有步骤不满足规则, 所以该SQL不满足规则
``` 

分步骤不稳定

但分步骤加上":"结尾, 会变得稳定: 

```
Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等 条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}:
ChatGLM-6B: 否否否否否否否否否否

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等 条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}:
ChatGLM-6B: 否否否否否否否否否否

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等 条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: 是是是是是是是是是是

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等 条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: 否否否否否否否否否否
``` 

将训练数据的instruction调整成带冒号的完整句子, 不用input. instruction如下: 

```
{AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='aaaa')), op=Keyword(text='and'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='='), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}:
``` 

这样训练, 不再会有因为冒号导致的不稳定.

但总步骤和分步骤仍然会有部分不一致: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='and'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 是是是是是是是是是是
   2. 结论: 所有检查均满足规则, 结论为该SQL满足规则

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='and'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}:
ChatGLM-6B: 否否否否否否否否否否
``` 

# 使用提示词来稳定输出

训练数据: 

```
判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT count(*) FROM head WHERE age  = age for update], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause; 否则, 则SQL不满足规则]:
思考步骤:
	1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 
		- 检查Select语句[1]. {AST}是否[直接使用]WhereClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='count'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Star(), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='head')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='age'), op=Punctuation(text='='), right=Identifier(text='age'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: 是是是是是是是是是是
	2. 结论: 所有检查均满足规则, 结论为该SQL满足规则
------
判断{SQL}是否符合{AST检查规则}: {SQL}: [{sql}], {AST检查规则}: [{ast_rule}]:
``` 

通过提示词给一组样例数据, 再问问题, 使得格式"变稳定"(?)

测试数据: 

```
判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT count(*) FROM head WHERE age  = age for update], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause; 否则, 则SQL不满足规则]:\n思考步骤:\n\t1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: \n\t\t- 检查Select语句[1]. {AST}是否[直接使用]WhereClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='count'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Star(), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='head')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='age'), op=Punctuation(text='='), right=Identifier(text='age'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: 是是是是是是是是是是\n\t2. 结论: 所有检查均满足规则, 结论为该SQL满足规则\n------\n判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用 的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]:\n
``` 

总步骤和分步骤变的一致, 且更加稳定

猜测: 对于总步骤, 在提示词中给入一组样例数据, 使得样例数据的总步骤-分步骤的关系更加被强调, 使得测试数据对于总-分关系的理解加强

测试数据2: 结果错误: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT count(*) FROM head WHERE age  = age for update], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause; 否则, 则SQL不满足规则]:\n思考步骤:\n\t1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: \n\t\t- 检查Select语句[1]. {AST}是否[直接使用]WhereClause: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='count'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Star(), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='head')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='age'), op=Punctuation(text='='), right=Identifier(text='age'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)}: 是是是是是是是是是是\n\t2. 结论: 所有检查均满足规则, 结论为该SQL满足规则\n------\n判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则]:\n

答：
思考步骤：

1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
- 检查Select语句[1]. {AST}直接使用的WhereClause中是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')), op=Keyword(text='>')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: 否否否否否否否否否否\n\t2. 结论: 有步骤不满足规则,  所以该SQL不满足规则\n------
 
 
-----
 
问：{AST}直接使用的WhereClause中是否[不包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')), op=Keyword(text='>')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}

答：
是是是是是是是是是是
``` 

总-分关系仍然不一致, 且不稳定

# 使用提示词来稳定输出[2]

修改提示词: 

```
判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT count(*) FROM head WHERE age  = age for update], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause; 否则, 则SQL不满足规则]:
思考步骤:
	1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 
		- 检查Select语句[1]. {AST}是否[直接使用]WhereClause: [填写{AST}]: [填写问题\"{AST}是否[直接使用]WhereClause\"的答案]
	2. 结论: [填写结论]
------
判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则]:
``` 

上一例效果可正常, 但稍微改变后, 就会出现 总-分 不一致:

```
Input: {AST}直接使用的WhereClause中是否[不包括]任何恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: prompt=[Round 1]

问：{AST}直接使用的WhereClause中是否[不包括]任何恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}

答：
否否否否否否否否否否
``` 

# 使用提示词来稳定输出[3]

增加"问题/答案"作为关键部分的提示, 效果有提升, 但仍会出现不一致: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a > 6 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: <是是是是是是是是是是>
   2. 结论: 所有检查均满足规则, 结论为该SQL满足规则

问：{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}

答：
否否否否否否否否否否
 
Input: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:
ChatGLM-6B: prompt=[Round 1]

问：问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
否否否否否否否否否否
``` 

逐步试探: 

```
问：思考步骤: \n 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
是是是是是是是是是是

----

问：xxxx: \n 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
否否否否否否否否否否

----

问：思考步骤: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
是是是是是是是是是是

----

问：xxxx: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
否否否否否否否否否否

----
 
问：思考步骤: 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
否否否否否否否否否否
 
----
 
问：思考步骤: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
是是是是是是是是是是
 
----

问：思考步骤: 1.           获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='>'), right=NumericLiteral(value='6'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
否否否否否否否否否否
``` 

![image2023-8-18 13:41:52.png](/assets/01KJBZ2E79S5TVATXJ30J7B9RZ/image2023-8-18%2013%3A41%3A52.png)

需要引入词汇变换震动, 来扛住前置符号带来的不确定性

使用 EDA (<https://github.com/zhanlaoban/eda_nlp_for_Chinese>) 引入震动, 使用参数, 增加一倍干扰数据: 

```
eda(input, alpha_sr=0, alpha_ri=alpha, alpha_rs=alpha, p_rd=0, num_aug=num_aug)
``` 

效果: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE 1=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
思考 步骤 : 1 . 获取 { SQL } 对应 的 { AST } , 并 使用 { AST 检查 规则 } 进行 检查 : - 问题 : < { AST } 是否 [ 不 直接 使用 ] WhereClause , 或者 { AST } 直接 使用 的 WhereClause 中 是否 [ 包括 ] 恒等 条件 : { Select ( with _ clause = None , modifiers = [ ] , select _ exprs = [ WithTrailingComma ( node = SelectExpr ( expr = Identifier ( text = ' dddd ' ) , alias = Identifier ( text = ' cccc ' ) ) , trailing _ comma = None ) ] , into _ before _ from = None , from _ clause = FromClause ( table = [ WithTrailingComma ( node = SimpleTableFactor ( table _ name = SimpleTableName ( identifier = Identifier ( text = ' a ' ) ) , as _ kw = None , alias = None , index _ hint _ list = [ ] ) , trailing _ comma = None ) ] ) , where = WhereClause ( conditions = BinOp ( left = NumericLiteral ( value = ' 1 ' ) , op = Punctuation ( text = ' = ' ) , right = NumericLiteral ( value = ' 1 ' ) ) ) , group _ by = None , having = None , order _ by = None , limit = None , into _ before _ lock _ mode = None , lock _ mode = None , into = None ) } > 答案 : < 是 是 是 是 是 是 是 是 是 是 > 2 . 结论 : 所有 检查 均 满足 规则 , 结论 为 该 SQL 满足 规则

---

问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE 1=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: <否否否否否否否否否否>
   2. 结论: 有步骤不满足规则, 所以该SQL不满足规则

---

问：{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}

答：
是 是 是 是 是 是 是 是 是 是

---

问：{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}

答：
是是是是是是是是是是
``` 

紧凑型的总步骤, 输出是错误的. 松散型的总步骤是正确的. 所有分步骤是正确的. 

也就是说, 紧凑型的输出是陷入了另外一套逻辑?

另一个案例: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
思考 步骤 : 1 . 获取 { SQL } 对应 的 { AST } , 并 使用 { AST 检查 规则 } 进行 检查 : - 问题 : < { AST } 是否 [ 不 直接 使用 ] WhereClause , 或者 { AST } 直接 使用 的 WhereClause 中 是否 [ 包括 ] 恒等 条件 : { Select ( with _ clause = None , modifiers = [ ] , select _ exprs = [ WithTrailingComma ( node = SelectExpr ( expr = Identifier ( text = ' dddd ' ) , alias = Identifier ( text = ' cccc ' ) ) , trailing _ comma = None ) ] , into _ before _ from = None , from _ clause = FromClause ( table = [ WithTrailingComma ( node = SimpleTableFactor ( table _ name = SimpleTableName ( identifier = Identifier ( text = ' a ' ) ) , as _ kw = None , alias = None , index _ hint _ list = [ ] ) , trailing _ comma = None ) ] ) , where = WhereClause ( conditions = BinOp ( left = Identifier ( text = ' a ' ) , op = Punctuation ( text = ' = ' ) , right = NumericLiteral ( value = ' 1 ' ) ) ) , group _ by = None , having = None , order _ by = None , limit = None , into _ before _ lock _ mode = None , lock _ mode = None , into = None ) } > 答案 : < 是 是 是 是 是 是 是 是 是 是 > 2 . 结论 : 所有 检查 均 满足 规则 , 结论 为 该 SQL 满足 规则

问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
思考 步骤 : 1 . 获取 { SQL } 对应 的 { AST } , 并 使用 { AST 检查 规则 } 进行 检查 : - 问题 : < { AST } 是否 [ 不 直接 使用 ] WhereClause , 或者 { AST } 直接 使用 的 WhereClause 中 是否 [ 包括 ] 恒等 条件 : { Select ( with _ clause = None , modifiers = [ ] , select _ exprs = [ WithTrailingComma ( node = SelectExpr ( expr = Identifier ( text = ' dddd ' ) , alias = Identifier ( text = ' cccc ' ) ) , trailing _ comma = None ) ] , into _ before _ from = None , from _ clause = FromClause ( table = [ WithTrailingComma ( node = SimpleTableFactor ( table _ name = SimpleTableName ( identifier = Identifier ( text = ' a ' ) ) , as _ kw = None , alias = None , index _ hint _ list = [ ] ) , trailing _ comma = None ) ] ) , where = WhereClause ( conditions = BinOp ( left = Identifier ( text = ' a ' ) , op = Punctuation ( text = ' = ' ) , right = NumericLiteral ( value = ' 1 ' ) ) ) , group _ by = None , having = None , order _ by = None , limit = None , into _ before _ lock _ mode = None , lock _ mode = None , into = None ) } > 答案 : < 否否 否否 否否 否否 否否 > 2 . 结论 : 有步骤 不 满足 规则 ,  所以 该 SQL 不 满足 规则
 
 
---
 
问：{AST}是否[不直接使用]WhereClause,或者{AST}直接使用的WhereClause中是否[包括]恒等条件:{Select(with_clause=None,modifiers=[],select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'),alias=Identifier(text='cccc')),trailing_comma=None)],into_before_from=None,from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')),as_kw=None,alias=None,index_hint_list=[]),trailing_comma=None)]),where=WhereClause(conditions=BinOp(left=Identifier(text='a'),op=Punctuation(text='='),right=NumericLiteral(value='1'))),group_by=None,having=None,order_by=None,limit=None,into_before_lock_mode=None,lock_mode=None,into=None)}

答：
是是是是是是是是是是

问：{AST}是否[不直接使用]WhereClause,或者{AST}直接使用的WhereClause中是否[包括]恒等条件:{Select(with_clause=None,modifiers=[],select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'),alias=Identifier(text='cccc')),trailing_comma=None)],into_before_from=None,from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')),as_kw=None,alias=None,index_hint_list=[]),trailing_comma=None)]),where=WhereClause(conditions=BinOp(left=Identifier(text='a'),op=Punctuation(text='='),right=NumericLiteral(value='1'))),group_by=None,having=None,order_by=None,limit=None,into_before_lock_mode=None,lock_mode=None,into=None)}

答：
否否否否否否否否否否
``` 

判断不稳定导致的总步骤不稳定

# 再次尝试

使用函数对用例随机增加空格, 将用例增加1倍: 

```
import random

def insert_spaces(text, num_positions, max_spaces):
    # 将字符串转换为列表，方便插入空格
    text_list = list(text)

    # 获取字符串长度和可插入空格的位置列表
    length = len(text_list)
    positions = random.sample(range(length), int(num_positions))

    # 在指定位置插入空格
    for pos in positions:
        spaces_to_insert = random.randint(0, max_spaces)
        text_list[pos] = ' ' * spaces_to_insert + text_list[pos]

    # 将列表转换回字符串
    result = ''.join(text_list)
    return result

def insert_spaces_list(text, num_positions, max_spaces, num_aug):
    ret = []
    for i in range(num_aug):
        ret.append(insert_spaces(text, num_positions, max_spaces))
    return ret
``` 

效果: 仍会出现总分步骤不一致, 但问题出在换行符? 

```
问：思考步骤: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: - 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则         : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='ddddd'), alias=Identifier(text='ccccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
否否否否否否否否否否

Input: 思考步骤: \n 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: - 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则         : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='ddddd'), alias=Identifier(text='ccccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}>
ChatGLM-6B: prompt=[Round 1]

问：思考步骤: \n 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查: - 问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则         : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='ddddd'), alias=Identifier(text='ccccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}>

答：
       是是是是是是是是是是
``` 

将训练数据的换行去除, 效果仍然不好: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT * FROM products AS products where 1 = 1], {AST检查规则}: [检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条 件; 否则, SQL不满足规则]

答：
思考步骤: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:         - 问题: <{AST}直接使用的WhereClause中是否[不包括]任何恒等条件        : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=Keyword(text='AS'), alias=Identifier(text='products'), index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: <是是是是是是是是是是>
   2. 结论: 所有检查均满足规则, 结论为该SQL满足规则

问：<{AST}直接使用的WhereClause中是否[不包括]任何恒等条件        : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=Keyword(text='AS'), alias=Identifier(text='products'), index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}>

答：
否否否否否否否否否否
``` 

探索: 

```
问：问: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:         - 问题: <{AST}直接使用的WhereClause中是否[不包括]任何恒等条件        : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=Keyword(text='AS'), alias=Identifier(text='products'), index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}>

答：
否否否否否否否否否否

---

问：步: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:         - 问题: <{AST}直接使用的WhereClause中是否[不包括]任何恒等条件        : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=Keyword(text='AS'), alias=Identifier(text='products'), index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}>

答：
否否否否否否否否否否

---

问：骤: 1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:         - 问题: <{AST}直接使用的WhereClause中是否[不包括]任何恒等条件        : {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='products')), as_kw=Keyword(text='AS'), alias=Identifier(text='products'), index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}>

答：
     是是是是是是是是是是
``` 

去除所有前置的词语, 仍然会发现总分不匹配: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
子问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: <是是是是是是是是是是>
结论: 所有检查均满足规则, 结论为该SQL满足规则

---

问：子问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}>

答：
否否否否否否否否否否

---

问：\n子问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案:

答：
否否否否否否否否否否
``` 

# 多轮微调

通过多轮微调进行反复学习

  - v1: explains_and_rules
  - v2: audits
  - v3: explains_and_rules
  - v4: audits

效果: 出现总步骤结果不稳定: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
子问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: <否否否否否否否否否否>
结论: 有步骤不满足规则, 所以该SQL不满足规则

---

问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
子问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: <是是是是是是是是是是>
结论: 所有检查均满足规则, 结论为该SQL满足规则
``` 

扩展到6轮训练, 总步骤结果稳定, 但总分不一致: 

```
问：判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a=1], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, 则SQL不满足规则]

答：
子问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: <是是是是是是是是是是>
结论: 所有检查均满足规则, 结论为该SQL满足规则

---

问：{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}

答：
否否否否否否否否否否否

---

问：子问题: <{AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[包括]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}> 答案: 

答：
否否否否否否否否否否
``` 

使用函数式定义, 会出现不稳定: 

```
问：sql='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: ast_rule[1]='''{AST}直接使用的WhereClause中是否[包括]恒等条件''', ast[1]='''Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)''', is_ast_match_ast_rule(ast[1], ast_rule[1])->'''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

Input: clear
History has been removed.

Input: sql='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?
ChatGLM-6B: prompt=[Round 1]

问：sql='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: ast_rule[1]='''{AST}直接使用的WhereClause中是否[包括]恒等条件''', ast[1]='''Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)''', is_ast_match_ast_rule(ast[1], ast_rule[1])->'''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
``` 

认为是总问题影响了分问题: 

```
问：ast_rule[1]='''{AST}直接使用的WhereClause中是否[包括]恒等条件''', ast[1]='''Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)''', is_ast_match_ast_rule(ast[1], ast_rule[1])

答：
否否否否否否否否否否

Input: ast_rule[1]='''{AST}直接使用的WhereClause中是否[包括]恒等条件''', ast[1]='''Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)''', is_ast_match_ast_rule(ast[1], ast_rule[1])->
ChatGLM-6B: prompt=[Round 1]

问：ast_rule[1]='''{AST}直接使用的WhereClause中是否[包括]恒等条件''', ast[1]='''Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)''', is_ast_match_ast_rule(ast[1], ast_rule[1])->

答：
是是是是是是是是是是
```
