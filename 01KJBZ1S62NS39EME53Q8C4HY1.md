---
title: 20230712 - chatglm 微调, 对sql进行规则审核
confluence_page_id: 2588839
created_at: 2023-07-12T07:31:39+00:00
updated_at: 2023-07-16T10:37:27+00:00
---

# 问题1

以下问答结论错误;

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from b order by c>, 规则: <select 语句不能使用order by>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用order by]:
    [检查AST中每一个Select和Subselect子句, 不能直接(不包括子查询)使用了OrderByClause; 否则, SQL不满足规则]
2. [获取SQL<>对应的AST: select * from b order by c]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='c')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - 步骤2中的Select的AST. AST<>是否直接(不包括子查询)使用了OrderByClause: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='c')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None): 否. 结论: 不满足规则
4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

对ast的规则判断有误; 

```
User: AST<>是否直接(不包括子查询)使用了OrderByClause: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='b')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='c')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)

ChatGLM-6B: 否
``` 

增加ast规则; 

```
explains.append({
    "instruction": "AST<>是否直接(不包括子查询)使用了OrderClause",
    "input": "Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=NumericLiteral(value='1')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)",
    "output": "是"
})

explains.append({
    "instruction": "AST<>是否直接(不包括子查询)使用了OrderClause",
    "input": "Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)",
    "output": "否"
})

``` 

与问题4的解决方案一样, 增加更多的AST操作判断, 可使结果正确

# 问题2

以上问答, 还有一个问题就是 "否" -> "不满足规则", 是匹配了训练数据中类似的一个规则 (limit的规则) 的问答, 而不适用于order的规则

当改变SQL, 使其偏离 相似的规则后, 结果修正:

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a where a in (select b from c order by t) limit 1>, 规则: <select 语句不能使用order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用order by]
        [检查AST中每一个Select语句, 不能直接(不包括子查询)使用OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a where a in (select b from c order by t) limit 1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleJoinedTable(left=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), inner_cross=None, join_kw=Keyword(text='IN'), right=SubqueryFactor(lateral_kw=None, table_subquery=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, limit_num=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), as_kw=None, alias=Identifier(text='t')), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, limit_num=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleJoinedTable(left=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), inner_cross=None, join_kw=Keyword(text='IN'), right=SubqueryFactor(lateral_kw=None, table_subquery=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, limit_num=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')), as_kw=None, alias=Identifier(text='t')), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, limit_num=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 是, 满足规则
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, limit_num=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则

``` 

但 以下SQL结果会错误: 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a where a in (select b from c order by t) limit 1>, 规则: <select 语句不能使用order by>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用order by]
        [检查AST中每一个Select语句, 不能直接(不包括子查询)使用OrderByClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from a where a in (select b from c order by t) limit 1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=SelectStmt(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=None, offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用OrderByClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=SelectStmt(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='t')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=None, offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))) )不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

# 问题3

测试规则推广: 

```
判断SQL是否符合 我们定义的SQL规则. SQL: <select c from d group by u having 1=1>, 规则: <select 语句不能使用Having子句>
``` 

增加规则的训练数据: 

```
rules.append({
    "instruction": "获取我们定义的SQL规则<>对应的 '检查规则'",
        "input": "select 语句不能使用Having子句",
    "output": "检查AST中每一个Select语句, 不能直接(不包括子查询)使用了HavingClause; 否则, SQL不满足规则"
})
``` 

效果: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select 1 from a limit 2>, 规则: <select 语句不能使用having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用having子句]
        [检查AST中每一个Select语句, 不能直接(不包括子查询)使用了HavingClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a limit 2]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='2'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否没有直接(不包括子查询)使用了HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='2'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

对规则的判断是反的, 增加更多的"没有直接使用"的训练数据?

在增加规则AST判断数据: 

```
rule = "AST<>是否不直接(不包括子查询)使用HavingClause"

explains.append(generate_explain_sample(rule,
    'select 1 from a group by b',
    True
))
explains.append(generate_explain_sample(rule,
    'select 1 from a group by b having 1=1',
    False
))
explains.append(generate_explain_sample(rule,
    'select 1 from a join (select c from d group by b having 1=1) b group by c',
    True
))
explains.append(generate_explain_sample(rule,
    'select 1 from a join (select c from d group by b having 1=1) b group by c having d=3',
    False
))
``` 

训练数据不足, 仍有错误: 

```
//select 1 from a limit 2

User: AST<>是否直接(不包括子查询)使用HavingClause: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='2'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)

ChatGLM-6B: 是
``` 

补充更多的训练数据, 可修订AST判断. 

碰到问题: 输出格式只给出了大纲, 不能匹配完整的输出模板

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a group by b having 1=1>, 规则: <select 语句不能使用Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则](获取我们定义的SQL规则)
    2. [获取SQL](获取SQL)
    3. [遍历SQL中的每一条规则>(遍历SQL中的每一条规则)
    4. [对于每条规则,检查其是否满足规则中的条件](对于每条规则,检查其是否满足规则中的条件)
    5. [如果满足条件,则输出“满足规则”](如果满足条件,则输出“满足规则”)
    6. [否则输出“不满足规则](否则输出“不满足规则”)

    步骤2中,获取规则时需要将规则中的<和>去掉

    步骤3中,遍历SQL中的每一条规则,需要使用for循环

    步骤4中,对于每条规则,需要使用if语句检查是否满足规则中的条件

    步骤5中,需要使用endif语句结束if语句
``` 

需要提高训练次数

仍存在问题,

## 问题1: "不直接(不包括子查询)" 和 "使用" 被分开理解了. 需要修订训练数据

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a group by b having 1=1>, 规则: <select 语句不能使用Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用Having子句]
        [检查AST中每一个Select语句, 不能直接(不包括子查询)使用Having; 否则, SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a group by b having 1=1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用Having: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None) : [不直接(不包括子查询)]
    4. 步骤3的AST<>是否满足步骤1的规则: [不满足规则]

    结论: 步骤3的AST不满足步骤1的规则
``` 

调整了训练数据中规则的分词: 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a group by b having 1=1>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a group by b having 1=1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=Identifier(text='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None) : 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

以上结果正确, 但以下结果错误: 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a group by b>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a group by b]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
 
---
 
User: AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]:

ChatGLM-6B: 是
``` 

猜测需要增加主流程的训练数据, 使得主流程中 "是/否" 的判断影响, 与AST的判断相关度更高, 不向主流程数据"过拟合"

## 问题2: 当SQL变化时, 输出步骤会有误, 增加 主流程 的训练数据?

````
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a group by b having 1=1>, 规则: <select 语句不能使用Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则](获取我们定义的SQL规则)
    2. [检查SQL是否符合规则](检查SQL是否符合规则)
        1. [检查[规则1](检查规则1)]
        2. [检查[规则2](检查规则2)]
    3. [输出结果](输出结果)

获取我们定义的SQL规则:
```less
SELECT * FROM a GROUP BY b HAVING 1 = 1;
```

检查SQL是否符合规则:
```sql
SELECT * FROM a GROUP BY b HAVING 1 = 1;
```

检查[规则1](检查规则1):
```sql
SELECT * FROM a GROUP BY b HAVING 1 = 1;
```
- 该SQL语句使用GROUP BY子句将结果分组。
- 该GROUP BY子句中包含SELECT语句。
- 不违反规则1。

检查[规则2](检查规则2):
```sql
SELECT * FROM a GROUP BY b HAVING 1 = 1;
```
- 不违反规则2。

结论:
该SQL语句符合我们定义的规则。

```` 

调整了AST规则的分词, 输出仍然有不准: 

会输出NotBinary的判断步骤. 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a group by b having 1=1>, 规则: <select 语句<不能使用>Having子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句<不能使用>Having子句]
        [检查AST中每一个Select语句, <不能直接(不包括子查询)使用>HavingClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select * from a group by b having 1=1]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否<不直接(不包括子查询)使用>HavingClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=GroupByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='b')), trailing_comma=None)]), having=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
        - 步骤2中的AST. AST<>是否满足规则: [NotBinary(left=None, op=Punctuation(text='='), right=None), right=NumericLiteral(value='1'), trailing_comma=None)]
        - 步骤2中的Select的AST. AST<>是否满足规则: [NotBinary(left=WithTrailingComma(node=Identifier(text='a')), op=Punctuation(text='='), right=NumericLiteral(value='b'), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=SelectExpr(expr=WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)), op=Punctuation(text='='), right=NumericLiteral(value='1'), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=OrderByExpr(expr=Identifier(text='b')), op=Punctuation(text='='), right=NumericLiteral(value='1'), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=GroupByExpr(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='a')), trailing_comma=None)]), order_by=None, limit=None), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=HavingClause(conditions=BinOp(left=NumericLiteral(value='1'), op=Punctuation(text='='), right=NumericLiteral(value='1'))), order_by=None, limit=None), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=OrderByExpr(expr=Identifier(text='b')), op=Punctuation(text='='), right=NumericLiteral(value='1'), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=WithTrailingComma(node=Identifier(text='a')), op=Punctuation(text='='), right=NumericLiteral(value='b'), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=SelectExpr(expr=WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)), op=Punctuation(text='='), right=NumericLiteral(value='1'), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='b'))]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=WithTrailingComma(node=Identifier(text='b')), op=Punctuation(text='='), right=NumericLiteral(value='1'), trailing_comma=None)]
        - 步骤2中AST<>是否满足规则: [NotBinary(left=Identifier(text='a'), op=Punctuation(text='='), right=NumericLiteral(value='b'))]
    4. 步骤3中AST<>是否满足规则: [步骤2中AST<>不满足规则]
``` 

# 问题4
    
    
    判断SQL是否符合 我们定义的SQL规则: SQL: , 规则: 

```
 
 
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a>, 规则: <select 语句不能使用limit子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用limit子句]
        [检查AST中每一个Select语句, 不能直接(不包括子查询)使用了SelectLimitClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
            - 步骤2中的Select的AST. AST<>是否没有直接(不包括子查询)使用了SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则

//
对步骤的检查规则是对的, 但对于规则的判断结果是错的

``` 

尝试增加对其他规则的正反判断:

  - 增加 "AST<>是否不直接(不包括子查询)使用SelectLimitClause" 的问答样例.
  - 但不增加 "判断SQL是否符合 我们定义的SQL规则: SQL: , 规则: " 的问答样例

可以达成正确的效果 (也就是说AST的判断问答, 会影响最终问答): 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用limit子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用limit子句]
        [检查AST中每一个Select语句, 不能直接(不包括子查询)使用SelectLimitClause; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a limit 10]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用SelectLimitClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
 
---
 
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a>, 规则: <select 语句不能使用limit子句>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用limit子句]
        [检查AST中每一个Select语句, 不能直接(不包括子查询)使用LIMIT; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select 1 from a]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用LIMIT: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]: 是, 满足规则
    4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

# 问题5

增加规则: select 语句不能使用SELECT ... INTO OUTFILE...

```
判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 INTO OUTFILE 'x' from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>
``` 

AST判断错误

```
User: AST<>是否直接(不包括子查询)使用IntoOutfile: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)

ChatGLM-6B: 是
``` 

给予针对性的训练数据:

```
rule = "AST<>是否直接(不包括子查询)使用IntoOutfile"

explains.append(generate_explain_sample(rule,
    'select * from a',
    False
))
explains.append(generate_explain_sample(rule,
    'select * into outfile "xxx" from a',
    True
))

explains.append(generate_explain_sample(rule,
    'select * from a where c=1',
    False
))
explains.append(generate_explain_sample(rule,
    'select * into outfile "xxx" from a where c=1',
    True
))
``` 

可修整.

之后遇到以下问题: 

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select ... into outfile...]
        [检查AST中每一个Select语句, 必须直接(不包括子查询)使用Outfile关键字; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select ... from a limit 10]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否直接(不包括子查询)使用Outfile关键字: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]: 否, 不满足规则
    4. 结论: 步骤3中有步骤不满足规则, 所以该SQL不满足规则
``` 

不能正确读取训练数据中的规则. 需要提高训练数据中的AST规则的占比.

提高后, 效果仍欠佳

```
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 INTO OUTFILE 'x' from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select ... INTO OUTFILE...>
        [检查AST中每一个Select语句, 必须直接(不包括子查询)使用IntoOutfile; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select ... from a limit 10]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用IntoOutfile: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None) : None
    4. 结论: 步骤3中没有不满足规则的AST, 所以步骤4为True, 结论为True
 
------
 
User: 判断SQL是否符合 我们定义的SQL规则: SQL: <select 1 from a limit 10>, 规则: <select 语句不能使用SELECT ... INTO OUTFILE...>

ChatGLM-6B: 思考步骤:
    1. [获取我们定义的SQL规则<> 对应的 '检查规则': select ... from a limit ...]
        [检查AST中每一个Select语句, 必须直接(不包括子查询)使用Select...Into Outfile...; 否则, 则SQL不满足规则]
    2. [获取SQL<>对应的AST: select ... from a limit ...]
        [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)
    3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
        - 步骤2中的Select的AST. AST<>是否不直接(不包括子查询)使用Select...Into Outfile...: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None) : SELECT ... FROM ... LIMIT ... FOR ...
    4. 步骤3. 该AST是否满足规则: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=NumericLiteral(value='1'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='1'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None) ... FROM ... LIMIT ... FOR ...]
        - 步骤3. 该规则不满足规则, 则步骤4为真, 结论为“不满足规则”
``` 

# 训练样例

```
(chatglm_etuning) root@r750-136:/data/huangyan/ChatGLM-Efficient-Tuning# less train.log | grep -A 1 'train_loss'
less: /root/miniconda3/lib/libtinfo.so.6: no version information available (required by less)
/bin/bash: /root/miniconda3/lib/libtinfo.so.6: no version information available (required by /bin/bash)
{'train_runtime': 2019.6322, 'train_samples_per_second': 4.159, 'train_steps_per_second': 0.99, 'train_loss': 0.03713942684233189, 'epoch': 190.48}
***** train metrics *****
--
  train_loss               =     0.0371
  train_runtime            = 0:33:39.63
--
{'train_runtime': 9601.2873, 'train_samples_per_second': 2.833, 'train_steps_per_second': 0.708, 'train_loss': 0.023181072464760612, 'epoch': 200.0}
***** train metrics *****
--
  train_loss               =     0.0232
  train_runtime            = 2:40:01.28
--
{'train_runtime': 5773.5005, 'train_samples_per_second': 3.083, 'train_steps_per_second': 0.762, 'train_loss': 0.001582831208678809, 'epoch': 98.88}
***** train metrics *****
--
  train_loss               =     0.0016
  train_runtime            = 1:36:13.50
```
```
(chatglm_etuning) root@r750-136:/data/huangyan/ChatGLM-Efficient-Tuning# less train.log | grep -A 1 'train_loss'
less: /root/miniconda3/lib/libtinfo.so.6: no version information available (required by less)
/bin/bash: /root/miniconda3/lib/libtinfo.so.6: no version information available (required by /bin/bash)
{'train_runtime': 504.0207, 'train_samples_per_second': 4.166, 'train_steps_per_second': 0.992, 'train_loss': 0.1493885978460312, 'epoch': 47.62}
***** train metrics *****
--
  train_loss               =     0.1494
  train_runtime            = 0:08:24.02
--
{'train_runtime': 2425.6212, 'train_samples_per_second': 2.803, 'train_steps_per_second': 0.701, 'train_loss': 0.09798823721268597, 'epoch': 50.0}
***** train metrics *****
--
  train_loss               =      0.098
  train_runtime            = 0:40:25.62
--
{'train_runtime': 1463.7294, 'train_samples_per_second': 3.04, 'train_steps_per_second': 0.752, 'train_loss': 0.002980233173478733, 'epoch': 24.72}
***** train metrics *****
--
  train_loss               =      0.003
  train_runtime            = 0:24:23.72
```
