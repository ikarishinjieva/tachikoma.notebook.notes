---
title: 20230717 - chatglm 微调, 对sql进行规则审核 [2]
confluence_page_id: 2588874
created_at: 2023-07-17T16:26:05+00:00
updated_at: 2023-07-22T16:13:08+00:00
---

# 背景

重构数据生成器, 只输入最终结果, 从中分离出每个步骤的训练数据

# 问题1

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句<不能使用>order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_
mClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma
e(conditions=BinOp(left=NumericLiteral(value='a'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], se
mma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor
e(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimi
iteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='a'), op
ight=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=
)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias
, trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), in
lock_mode=None, into=None), where=WhereClause(conditions=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None))
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=N
alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifi
 alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='a'), op=Punctuation(text='in'), right=Subselect(left_par
select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=No
(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]
ne, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=No
ditions=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None))): 否, 不满足规则
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=I
=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(te
s=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=
nto_before_lock_mode=None, lock_mode=None, into=None)]: 是, 满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

存在几个问题: 

  - 总AST 和 分AST[0] 不一致: 猜测是IN语句样例不足
  - 对于AST分析的判断 反了: 猜测是过拟合
  - AST中, 并没有解出 order_by: 猜测是IN语句样例不足

进行如下调整:

  - 丰富order_by的case, 引入IN语句样例
  - 划分验证集, 解决过拟合问题

验证效果: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句<不能使用>order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='a'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='a'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则

    ---

    User: AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=NumericLiteral(value='a'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
``` 

问题: 

  - 未分离子查询
  - 没能解析出 order_by
  - AST分析的结果错误, 与单独分析不一致

在order by规则中, 增加更多训练数据. 并且修改子查询的关键词. 效果: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句<不能使用>order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
        - 检查Select语句[2]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 是, 满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

问题: 

  - AST判断有误, 与单独的AST判断的结果相反

按照下面章节 "尝试修正 AST规则 不正确", 修正 "是/否" 的描述, 增大训练数据集, 结果回复正常

缩小训练结果集, 仍会出现AST判断误差

增加训练结果集, 仍会出现AST判断错误

# 问题2

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from b limit 9>, 规则: <select 语句<不能使用>order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from b limit 9]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

  - AST分析正确, 但放在总分析里错误

```
AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
``` 

猜测是过拟合, 与问题1解法类似, 解决过拟合的问题

引入验证集后, 已修复

增加一些数据后, 发生问题: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from b limit 9>, 规则: <select 语句<不能使用>order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from b limit 9]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='*')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='*')), trailing_comma=None)]), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

  - AST解析错误, 出现了order_by
  - 两段AST不一致, 第二段丢失了limit段
  - 单独的AST是正确的: 

```
User: 获取SQL<>对应的AST: select * from b limit 9

ChatGLM-6B: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), with_clause=None, into_lock_mode=None)
```

  - 尝试在order by规则中, 增加 select * + limit的组合

增加一些 select * + limit的组合后, 效果: 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from b limit 9>, 规则: <select 语句<不能使用>order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from b limit 9]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

修正了一些问题. 但AST判断仍然出错, 单独进行AST判断会输出正确结果

按照下面章节 "尝试修正 AST规则 不正确", 修正 "是/否" 的描述, 结果回复正常

# 问题3

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select c from d group by u having 1=1>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>Having子句; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select c from d group by u having 1=1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=Identifier(text='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>Having子句: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=Identifier(text='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则

---

User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select c from d group by u limit 1,2>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>Having子句; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select c from d group by u limit 1,2]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>Having子句: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

  - 检查规则 错误: 应该是HavingClause

  - AST判断 错误: 

```
User: AST<>是否<不直接(不包括子查询)使用>Having子句: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=Identifier(text='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]

ChatGLM-6B: 是
```

    - 但以下问答是正确的: 
    - (尝试其他正反用例, 答案均是正确的)

```
User: AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=Identifier(text='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]

ChatGLM-6B: 否
``` 

尝试解决: 

  - 引入一个额外的AST规则, 明确是HavingClause规则

效果: 结果正确, 但第二个规则的判断不稳定, 有时会出现AST判断与单独AST判断不同

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select c from d group by u limit 1,2>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select c from d group by u limit 1,2]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='2'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='2'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 是, 满足规则
    4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则

---

User: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT 1 FROM categories c LEFT JOIN products p ON c.id = p.category_id GROUP BY c.name HAVING product_count < 5>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select c from d group by u having 1=2]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='u')), trailing_comma=None)]), having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

增加训练数据后, 发生向SelectClause过拟合: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select c from d group by u having 1=1>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>SelectClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select c from d group by u having 1=1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Dotted(left=Identifier(text='c'), dot=Punctuation(text='.'), right=Identifier(text='from')), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=Dotted(left=Identifier(text='d'), dot=Punctuation(text='.'), right=Identifier(text='to')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Dotted(left=Identifier(text='u'), dot=Punctuation(text='.'), right=Identifier(text='u'))), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>SelectClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Dotted(left=Identifier(text='c'), dot=Punctuation(text='.'), right=Identifier(text='from')), alias=None), trailing_comma=Punctuation(text=',')), WithTrailingComma(node=SelectExpr(expr=Dotted(left=Identifier(text='d'), dot=Punctuation(text='.'), right=Identifier(text='to')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='d')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Dotted(left=Identifier(text='u'), dot=Punctuation(text='.'), right=Identifier(text='u'))), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

# 问题4

检查规则 错误, 向SelectLimitClause过拟合

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a group by b having 1=1>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>SelectLimitClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from a group by b having 1=1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1=1'), op=Punctuation(text='='), right=Identifier(text='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1=1'), op=Punctuation(text='='), right=Identifier(text='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

尝试解决: 引入校验集, 效果没有改观.

修复问题3后 (引入一个明确的AST规则), 该情况消失

增加训练数据后, 发生向SelectClause过拟合: 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a group by b having 1=1>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>Subselect; 否则, SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from a group by b having 1=1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>Subselect: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 是
    4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

增加AST rule的训练数据比例后, 解决过拟合, 但AST判断错误: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a group by b having 1=1>, 规则: <select 语句<不能使用>Having子句>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select * from a group by b having 1=1]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_fr
om=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias
=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), tr
ailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_
by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=Sele
ctExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_na
me=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(e
xpr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'
), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
 
---
 
Input: AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), a
lias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(id
entifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComm
a(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(tex
t='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
ChatGLM-6B: 否
``` 

# 引入校验集

引入校验集后, 可以观察到过拟合的现象: 

```
(chatglm_etuning) root@r750-136:/data/huangyan/ChatGLM-Efficient-Tuning# ls -alh -d checkpoint.sql_execution_plan_and_audit.v*/checkpoint-*
drwxr-xr-x 2 root root 255 Jul 17 17:41 checkpoint.sql_execution_plan_and_audit.v1/checkpoint-100
drwxr-xr-x 2 root root 255 Jul 17 17:43 checkpoint.sql_execution_plan_and_audit.v1/checkpoint-200
drwxr-xr-x 2 root root 255 Jul 17 17:46 checkpoint.sql_execution_plan_and_audit.v1/checkpoint-300
drwxr-xr-x 2 root root 255 Jul 17 17:48 checkpoint.sql_execution_plan_and_audit.v1/checkpoint-400
drwxr-xr-x 2 root root 215 Jul 17 17:51 checkpoint.sql_execution_plan_and_audit.v1/checkpoint-500
drwxr-xr-x 2 root root 215 Jul 17 17:53 checkpoint.sql_execution_plan_and_audit.v1/checkpoint-600
drwxr-xr-x 2 root root 255 Jul 17 17:59 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-100
drwxr-xr-x 2 root root 255 Jul 17 18:36 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1000
drwxr-xr-x 2 root root 255 Jul 17 18:40 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1100
drwxr-xr-x 2 root root 255 Jul 17 18:44 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1200
drwxr-xr-x 2 root root 255 Jul 17 18:48 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1300
drwxr-xr-x 2 root root 255 Jul 17 18:52 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1400
drwxr-xr-x 2 root root 255 Jul 17 18:57 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1500
drwxr-xr-x 2 root root 255 Jul 17 19:01 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1600
drwxr-xr-x 2 root root 255 Jul 17 19:05 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1700
drwxr-xr-x 2 root root 255 Jul 17 19:09 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1800
drwxr-xr-x 2 root root 255 Jul 17 19:13 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-1900
drwxr-xr-x 2 root root 255 Jul 17 18:03 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-200
drwxr-xr-x 2 root root 255 Jul 17 19:17 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-2000
drwxr-xr-x 2 root root 255 Jul 17 19:22 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-2100
drwxr-xr-x 2 root root 255 Jul 17 18:07 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-300
drwxr-xr-x 2 root root 255 Jul 17 18:11 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-400
drwxr-xr-x 2 root root 255 Jul 17 18:15 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-500
drwxr-xr-x 2 root root 255 Jul 17 18:19 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-600
drwxr-xr-x 2 root root 255 Jul 17 18:24 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-700
drwxr-xr-x 2 root root 255 Jul 17 18:28 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-800
drwxr-xr-x 2 root root 255 Jul 17 18:32 checkpoint.sql_execution_plan_and_audit.v2/checkpoint-900
drwxr-xr-x 2 root root 215 Jul 17 19:28 checkpoint.sql_execution_plan_and_audit.v3/checkpoint-100
drwxr-xr-x 2 root root 255 Jul 17 19:32 checkpoint.sql_execution_plan_and_audit.v3/checkpoint-200
drwxr-xr-x 2 root root 255 Jul 17 19:36 checkpoint.sql_execution_plan_and_audit.v3/checkpoint-300
drwxr-xr-x 2 root root 255 Jul 17 19:40 checkpoint.sql_execution_plan_and_audit.v3/checkpoint-400
drwxr-xr-x 2 root root 255 Jul 17 19:44 checkpoint.sql_execution_plan_and_audit.v3/checkpoint-500
drwxr-xr-x 2 root root 215 Jul 17 19:48 checkpoint.sql_execution_plan_and_audit.v3/checkpoint-600
 
 
/bin/bash: /root/miniconda3/lib/libtinfo.so.6: no version information available (required by /bin/bash)
07/17/2023 17:54:33 - INFO - utils.peft_trainer - Loading best model from checkpoint.sql_execution_plan_and_audit.v1/checkpoint-300 (score: 0.03975478932261467).
07/17/2023 19:24:23 - INFO - utils.peft_trainer - Loading best model from checkpoint.sql_execution_plan_and_audit.v2/checkpoint-400 (score: 0.4046516418457031).
07/17/2023 19:51:06 - INFO - utils.peft_trainer - Loading best model from checkpoint.sql_execution_plan_and_audit.v3/checkpoint-600 (score: 0.049982525408267975).
``` 

其中v1和v2都过拟合了, 选取了300/400两个checkpoint作为最优解

# 尝试修正 AST规则 不正确

现象: 在 总步骤 中, AST规则的判断, 与 单独进行AST规则判断时会不同. 举例: 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from b limit 9>, 规则: <select 语句<不能使用>order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from b limit 9]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
 
---
 
User: AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='9'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]

ChatGLM-6B: 是
``` 

尝试修改训练数据, 将 "否, 不满足规则" 改成 "否", "是, 满足规则" 改成 "是"

同时增加训练数据, 结果可恢复正常.

当减少训练数据, 结果又异常

# 问题5

测试规则推广性: 

```
判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 INTO OUTFILE 'x' from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
```
```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用SELECT ... INTO OUTFILE...]
       [检查AST中每一个Select语句, <>IntoOutfileClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a limit 10]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=Non
e)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a'))
, as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_coun
t=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<>IntoOutfileClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Numeric
Literal(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_na
me=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=No
ne, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, i
nto=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
 
---
 
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 INTO OUTFILE 'x' from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用SELECT ... INTO OUTFILE...]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用> into  OutfileClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 INTO OUTFILE 'x' from a limit 10]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=Non
e)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a'))
, as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_coun
t=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OutfileClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=Sel
ectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=Simple
TableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, grou
p_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=Non
e, lock_mode=None, into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

生成AST规则错误

# 问题6

训练新规则: select 语句<不能使用>for update

其规则判断语句会有所不同: "检查AST中每一个Select语句, 直接(不包括子查询)使用的LockMode中 不能包括FOR UPDATE关键字; 否则, SQL不满足规则"

是判断元素中的值, 尝试训练看看

```
判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT 1 from a FOR UPDATE>, 规则: <select 语句<不能使用>for update>
 
判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT SUM(quantity) FROM inventory WHERE product_id = 12345 FOR UPDATE>, 规则: <select 语句<不能使用>for update>
``` 

再次出现 总判断和AST判断不一致的情况: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT 1 from a FOR UPDATE>, 规则: <select 语句<不能使用>for update>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>for update]
       [检查AST中每一个Select语句, 直接(不包括子查询)使用的LockMode中<不能使用>FOR UPDATE关键字; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT 1 from a FOR UPDATE]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的LockMode中 是否<不使用>FOR UPDATE关键字: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则

Input: AST<>直接(不包括子查询)使用的LockMode中 是否<不使用>FOR UPDATE关键字: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
ChatGLM-6B: 否

Input: AST<>直接(不包括子查询)使用的LockMode中 是否<使用>FOR UPDATE关键字: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
ChatGLM-6B: 是
```

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT SUM(quantity) FROM inventory WHERE product_id = 12345 FOR UPDATE>, 规则: <select 语句<不能使用>for update>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>for update]
       [检查AST中每一个Select语句, 直接(不包括子查询)使用的LockMode中<不能使用>FOR UPDATE关键字; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT SUM(quantity) FROM inventory WHERE product_id = 12345 FOR UPDATE]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=NumericLiteral(value='12345'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的LockMode中 是否<不使用>FOR UPDATE关键字: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=NumericLiteral(value='12345'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则

Input: AST<>直接(不包括子查询)使用的LockMode中 是否<不使用>FOR UPDATE关键字: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=NumericLiteral(value='12345'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
ChatGLM-6B: 否

Input: AST<>直接(不包括子查询)使用的LockMode中 是否<使用>FOR UPDATE关键字: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=FunctionCall(callee=Identifier(text='SUM'), left_paren=Punctuation(text='('), args=[WithTrailingComma(node=Identifier(text='quantity'), trailing_comma=None)], right_paren=Punctuation(text=')')), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='inventory')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='product_id'), op=Punctuation(text='='), right=NumericLiteral(value='12345'))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
ChatGLM-6B: 是
``` 

将 sql_to_ast_datas 和 sql_rule_to_ast_rule_datas 数据增值, 形成以下数量

```
sql_to_ast_datas: 676
sql_rule_to_ast_rule_datas: 640
ast_rule_match_datas: 2704
audit_datas: 2096
``` 

结果仍然错误

# 问题7

新规则: select 语句<不能使用>LIMIT...OFFSET...

其中包含 LIMIT...OFFSET 和 LIMIT M,N 两种形式

再次出现 总判断和AST判断不一致的情况: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <SELECT 1 from a FOR UPDATE>, 规则: <select 语句<不能使用>LIMIT...OFFSET...>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>LIMIT...OFFSET...]
       [检查AST中每一个Select语句, 直接(不包括子查询)使用的SelectLimitClause中 offset<必须等于>None; 否则, SQL不满足规则]
   2. [获取SQL<>对应的AST: SELECT 1 from a FOR UPDATE]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>直接(不包括子查询)使用的SelectLimitClause中 offset<是否等于>None: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]: 否
   4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则

Input: AST<>直接(不包括子查询)使用的SelectLimitClause中 offset<是否等于>None: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
ChatGLM-6B: 是

Input: AST<>直接(不包括子查询)使用的SelectLimitClause中 offset<是否不等于>None: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=LockMode(mode=KeywordSequence(keywords=[Keyword(text='FOR'), Keyword(text='UPDATE')]), modifier=None), into=None)]
ChatGLM-6B: 否
``` 

将 sql_to_ast_datas 和 sql_rule_to_ast_rule_datas 数据增值, 形成以下数量

```
sql_to_ast_datas: 676
sql_rule_to_ast_rule_datas: 640
ast_rule_match_datas: 2704
audit_datas: 2096
``` 

现象得以修正

# 问题8

增加规则: select 语句 <不能使用> 无意义的where

与 "select语句<必须使用>where子句" 形成联合使用

# 问题9

```
Input: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句<不能使用>order by>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='t'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Punctuation(text='in'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 是
       - 检查Select语句[2]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='t'), alias=None), trailing_comma=None)]), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='t'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

对于order by的AST生成有误 (目前仅使用第一轮训练的结果, 等待第二轮训练后再观测)

第二轮训练后, order_by生成的位置有误: 

```
Input: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句<不能使用>order by>
ChatGLM-6B: 思考步骤:
   1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>order by]
       [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>OrderByClause; 否则, 则SQL不满足规则]
   2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
       [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), group_by=None, having=None, order_by=None, limit=None), into_before_lock_mode=None, lock_mode=None, into=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
   3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
       - 检查Select语句[1]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), group_by=None, having=None, order_by=None, limit=None), into_before_lock_mode=None, lock_mode=None, into=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 是
       - 检查Select语句[2]. AST<>是否<不直接(不包括子查询)使用>OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 是
   4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

猜测: 训练数据中, 子查询中含有order by的用例很少

将 sql_to_ast_datas 和 sql_rule_to_ast_rule_datas 数据增值, 现象得以修正

# 效率对比

  1. lora训练后的模型, 比p-tuning后的模型, 预测速度快
  2. p-tuning训练过程略快
