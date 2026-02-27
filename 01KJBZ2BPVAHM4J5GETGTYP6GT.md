---
title: 20230805 - chatglm 微调, 对sql进行规则审核 - 增加规则
confluence_page_id: 2588993
created_at: 2023-08-05T07:45:06+00:00
updated_at: 2023-08-07T16:13:23+00:00
---

# 问题1

```
Input: 获取{SQL规则}对应的{AST检查规则}: select 语句的where条件[不能仅使用]恒等条件
ChatGLM-6B: 检查每一个Select语句, {AST}中WhereClause中是否[使用(IMPORTANT_START)]恒等[IMPORTANT_END]检查, 否则[IMPORTANT_START]检查[IMPORTANT_END]不满足规则;

Input: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]LIMIT...OFFSET...
ChatGLM-6B: 检查每一个Select语句, {AST}直接使用的SelectLimitClause中 offset是否正确, 如果是[IMPORTANT_START]offset不正确, ImportedModules.CreateTable(table_name)中 offset不正确[IMPORTANT_END]
``` 
    
    
    获取{SQL规则}对应的{AST检查规则} 结果不准. 
    
    
    TODO: 提高样本量: 832

```
sql_to_ast_datas: 992
sql_rule_to_ast_rule_datas: 832
ast_rule_match_datas: 4464
audit_datas: 7200
``` 

仍然会出现不准. 

继续增加样本量

```
sql_to_ast_datas: 992
sql_rule_to_ast_rule_datas: 3328
ast_rule_match_datas: 8928
audit_datas: 7200
``` 

效果: 

```
Input: 获取{SQL规则}对应的{AST检查规则}: select 语句的where条件[不能仅使用]恒等条件
ChatGLM-6B: 检查每一个Select语句, {AST}直接使用的WhereClause中是否[不仅使用]恒等条件; 否则, SQL不满足规则

Input: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]LIMIT...OFFSET...
ChatGLM-6B: 检查每一个Select语句, 并{AST}不{禁止使用}LIMIT...OFFSET...之一, 则对应的SQL规则为: 选择语句[不禁止使用]LIMIT...OFFSET...
``` 

LIMIT...OFFSET的结果不稳定, 尝试将...符号去掉, 再次训练

将...符号去掉, 结果稳定 (怀疑与跟embedding的方法有关): 

```
Input: 获取{SQL规则}对应的{AST检查规则}: select 语句[不能使用]LIMIT OFFSET
ChatGLM-6B: 检查每一个Select语句, {AST}直接使用的SelectLimitClause中 offset是否[等于]None; 否则, SQL不满足规则

Input: 获取{SQL规则}对应的{AST检查规则}: select 语句的where条件[不能仅使用]恒等条件
ChatGLM-6B: 检查每一个Select语句, {AST}直接使用的WhereClause中是否[不仅使用]恒等条件; 否则, SQL不满足规则
``` 

# 问题2

```
Input: {AST}直接使用的WhereClause中 是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Star(), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='station')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='city'), op=Punctuation(text='='), right=Identifier(text='city'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|否|IMPORTANT_END|

Input: {AST}直接使用的WhereClause中 是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Star(), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='station')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='city'), op=Punctuation(text='='), right=Identifier(text='city'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|
``` 

rule匹配数据不稳定

增加训练数据 (但增加的是其他规则的训练数据, 原因不明), 结果变稳定但错误: 

```
Input: {AST}直接使用的WhereClause中 是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='COUNT'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Star(), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='station')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='city'), op=Punctuation(text='='), right=Identifier(text='city'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|
``` 

# 问题3.1

增加"并且"规则1: select语句[必须使用]where, 并且where条件[不能仅使用]恒等条件

增加规则2: 别名[不要]与表或列的名字相同

```
Input: 获取{SQL规则}对应的{AST检查规则}: select语句[必须使用]where, 并且where条件[不能仅使用]恒等条件
ChatGLM-6B: 检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件; 否则, 则SQL不满足规则

Input: 获取{SQL规则}对应的{AST检查规则}: select语句[必须使用]where, 并且where条件[不能仅使用]恒等条件
ChatGLM-6B: 检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件; 否则, 则SQL不满足规则

Input: 获取{SQL规则}对应的{AST检查规则}: select语句[必须使用]where, 并且where条件[不能仅使用]恒等条件
ChatGLM-6B: 检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件; 否则, 则SQL不满足规则

Input: 获取{SQL规则}对应的{AST检查规则}: 别名[不要]与表或列的名字相同
ChatGLM-6B: 检查每一个语句, 是否满足规则 {SQL规则} 对应的 {AST检查规则}
如果满足规则, 则输出 [IMPORTANT_START]有意义的输出 [IMPORTANT_END]
不满足规则, 则输出 [ERROR_START]有意义的输出 [ERROR_END]
满足规则, 则输出 [INFO_START]有意义的输出 [INFO_END]

Input: 获取{SQL规则}对应的{AST检查规则}: 别名[不要]与表或列的名字相同
ChatGLM-6B: 检查每一个语句, 是否使用 否, 或者 0 或者 None; 否则, 则认为是 SQL 规则 对应的AST检查规则。
``` 

规则1稳定, 规则2不稳定

探索: [不能] 关键词会导致错误, [必须] 是正确的. 

```
Input: 获取{SQL规则}对应的{AST检查规则}:别名[必须]与表或列的名字相同
ChatGLM-6B: 检查每一个语句, {AST}的alias, 是否与相关的table_name或expr[相同]; 否则, 则SQL不满足规则

Input: 获取{SQL规则}对应的{AST检查规则}:别名[不能]与表或列的名字相同
ChatGLM-6B: 检查每一个语句, {AST}的alias, 是否[与]表达式或[相同]在同一个语句中。 否则, 则SQL不满足规则。
``` 

暂时修改规则描述, 尝试绕过

使用修改后的AST检查规则, 仍然会有偏差: 

```
Input: 获取{SQL规则}对应的{AST检查规则}: 别名必须与表或列的名字[不相同]
ChatGLM-6B: 检查每一个语句, 并取其中的一部分, 检查是否与之前检查的规则相同, 如果不相同, 则返回False; 否则, 继续检查下一个语句。 返回True, 则该规则对应的AST检查规则成立。

Input: 获取{SQL规则}对应的{AST检查规则}: 别名必须与表或列的名字[相同]
ChatGLM-6B: 检查每一个语句, 并获取其中的{AST检查规则}和{SQL规则}, 然后获取对应的{AST检查规则}和{SQL规则}是否满足所要求的{SQL规则} 或{AST检查规则}
``` 

两点变更: 

  1. 为AST检查规则的获取, 增加IMPORTANT标签
  2. 并将总步骤(简版)中的IMPORTANT标签去掉 (避免交叉影响)

效果正确: 

```
Input: 获取{SQL规则}对应的{AST检查规则}: 别名必须与表或列的名字[不相同]
ChatGLM-6B: [IMPORTANT_START]检查每一个语句, {AST}的每一个alias, 是否与相关的table_name或expr[不相同]; 否则, 则SQL不满足规则[IMPORTANT_END]

Input: 获取{SQL规则}对应的{AST检查规则}: 别名必须与表或列的名字[相同]
ChatGLM-6B: [IMPORTANT_START]检查每一个语句, {AST}的每一个alias, 是否与相关的table_name或expr[相同]; 否则, 则SQL不满足规则[IMPORTANT_END]
``` 

猜测是变更2发挥了作用, 尝试将变更1回滚 (去掉AST检查规则的IMPORTANT标签)

将变更1回滚后, 效果仍然能保持

结论: 简版的总步骤的训练数据中, 如果带有IMPORTANT提高权重, 会影响到AST检查规则的训练

# 问题3.2

规则1的判断结果不正确: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|否|IMPORTANT_END|
``` 

规则2 总步骤和分步骤再次不一致: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT c FROM a as a WHERE a = a], {AST检查规则}: [检查每一个语句, {AST}的alias, 是否与相关的table_name或expr[不相同]; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}的alias, 与相关的table_name或expr是否[不相同]; 否则, 则SQL不满足规则: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=Keyword(text='AS'), alias=Identifier(text='a'), index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: {AST}的alias, 与相关的table_name或expr是否[不相同]; 否则, 则SQL不满足规则: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=Keyword(text='AS'), alias=Identifier(text='a'), index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|
 
 
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT c FROM a WHERE a = a], {AST检查规则}: [检查每一个语句, {AST}的alias, 是否与相关的table_name或expr[不相同]; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}的alias, 与相关的table_name或expr是否[不相同]; 否则, 则SQL不满足规则: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input:  {AST}的alias, 与相关的table_name或expr是否[不相同]; 否则, 则SQL不满足规则: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='c'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|
 
 
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as dddd FROM a WHERE a = a], {AST检查规则}: [检查每一个语句, {AST}的alias, 是否与相关的table_name或expr[不相同]; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}的alias, 与相关的table_name或expr是否[不相同]; 否则, 则SQL不满足规则: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='dddd')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: {AST}的alias, 与相关的table_name或expr是否[不相同]; 否则, 则SQL不满足规则: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='dddd')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='a'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|否|IMPORTANT_END|
``` 

尝试增加反向数据, 仍然会有不一致: 

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT 1 FROM a WHERE a = b and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='b')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=Identifier(text='b')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|
``` 

IMPORTANT权重调整到2000

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅 使用]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|否|IMPORTANT_END|
``` 
    
    
    AST判断会不稳定, 转为问题5

# 问题5

总步骤和AST判断间, 刚好相反

```
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅 使用]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]否[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]有步骤不满足规则, 所以该SQL不满足规则[IMPORTANT_END]

Input: {AST}是否[直接使用]WhereClause, 并且{AST}直接使用的WhereClause中是否[不仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|

Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT dddd as cccc FROM a WHERE a = 1 and b > 2], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[仅 使用]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]

Input: {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='dddd'), alias=Identifier(text='cccc')), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='>'), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|否|IMPORTANT_END|
 
且AST判断并不稳定: 
 
Input: 判断{SQL}是否符合{AST检查规则}: {SQL}: [SELECT * FROM aaa WHERE a = 1 and b = 2 or c = c], {AST检查规则}: [检查每一个Select语句, {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[仅 使用]恒等条件; 否则, 则SQL不满足规则]
ChatGLM-6B: 思考步骤:
   1. 获取{SQL}对应的{AST}, 并使用{AST检查规则}进行检查:
       - 检查Select语句[1]. {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='aaa')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='='), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}: [IMPORTANT_START]是[IMPORTANT_END]
   2. 结论: [IMPORTANT_START]所有检查均满足规则, 结论为该SQL满足规则[IMPORTANT_END]

Input: {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='aaa')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='='), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|是|IMPORTANT_END|

Input: {AST}是否[不直接使用]WhereClause, 或者{AST}直接使用的WhereClause中是否[仅使用]恒等条件: {Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='aaa')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=BinOp(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='1')), op=Keyword(text='AND'), right=BinOp(left=Identifier(text='b'), op=Punctuation(text='='), right=NumericLiteral(value='2')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)}
ChatGLM-6B: |IMPORTANT_START|否|IMPORTANT_END|
``` 

增加了多种IMPORTANT权值: 

  - 将总步骤中的 AST判断的 是/否 权值提升到 IMPORTANT_3
  - 将AST判断的训练数据, 结果中的 是/否 权值提升到 IMPORTANT

不一致的现象消除, 但对于"恒等条件"的判断是错误的
