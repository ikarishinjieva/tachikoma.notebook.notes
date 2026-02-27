---
title: 20230707 - chatglm微调, 使用SQL AST, 增强SQL规则验证(2)
confluence_page_id: 2588805
created_at: 2023-07-06T22:19:20+00:00
updated_at: 2023-07-07T17:57:49+00:00
---

# 20230707 进行过一次训练

训练结果: 

```
{'train_runtime': 13963.3245, 'train_samples_per_second': 2.134, 'train_steps_per_second': 0.53, 'train_loss': 0.017071028555164465, 'epoch': 198.66}
100%|
 
7400/7400 [3:52:43<00:00,  1.89s/it]
***** train metrics *****
  epoch                    =     198.66
  train_loss               =     0.0171
  train_runtime            = 3:52:43.32
  train_samples_per_second =      2.134
  train_steps_per_second   =       0.53
07/06/2023 19:46:20 - INFO - utils.peft_trainer - Saving model checkpoint to path_to_sft_checkpoint.sql_execution_plan_and_audit.v1
``` 

错误的结果: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from vvvv where ccccc in (select tttt from xxxx limit 5000)>, 规则: <select语句必须使用where条件>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select语句必须使用where条件]:
    [检查AST中每一个Select和Subselect子句, 必须使用WhereClause;否则, SQL不满足规则]
2. [获取SQL<>对应的AST: select * from vvvv where ccccc in (select tttt from xxxx limit 5000)]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='vvvv')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='ccccc'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - 问题: [步骤2中的Select的AST] 是否直接(不包括子查询)使用了WhereClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='vvvv')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='ccccc'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]. 答案: 是. 结论: 满足规则
    - 问题: [步骤2中的Subselect的AST] 是否直接(不包括子查询)使用了WhereClause: [Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))]. 答案: 是. 结论: 满足规则
4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
``` 

对于子句的判断是错误的, 单独验证两个子句: 

第一子句:

```
在完整答案中, 结果为"是"
---
 
User: [步骤2中的Select的AST] 是否直接(不包括子查询)使用了WhereClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='vvvv')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='ccccc'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]. 答案:

ChatGLM-6B: 否.
---
 
User: AST<>是否直接(不包括子查询)使用了WhereClause: [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='vvvv')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='ccccc'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]

ChatGLM-6B: 是
 
``` 

第二子句: 

```
在完整答案中, 结果为"是"
 
---
 
User: 问题: [步骤2中的Subselect的AST] 是否直接(不包括子查询)使用了WhereClause: [Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))]. 答案:

ChatGLM-6B: 否
 
---
 
User: AST<>是否直接(不包括子查询)使用了WhereClause: [Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')'))]

ChatGLM-6B: 是

``` 

尝试将第二子句的错误修正, 补充相关AST判断的数据

# 尝试增量训练

将"AST<>"数据补充一些, 作为增量训练的数据. 

无论resume_lora_training是否开启, 增量训练epoch>=1时, 原本的问题

"判断SQL是否符合 我们定义的SQL规则. SQL: , 规则: "

就会失去结构

询问chatgpt: 

```
如果在第二次微调中只使用问题类型B的数据，而不包括问题类型A的数据，那么模型很可能会调整自身以适应新的训练数据。这可能导致问题类型A的回答结果发生变化。
如果你希望在第二次微调中保持问题类型A的回答效果，有几种方法可以尝试：
固定问题类型A的相关层：在第二次微调期间，将问题类型A相关的模型层设置为不可训练或固定。这样，这些层的权重将保持不变，不会受到问题类型B数据的影响。

采用迁移学习的方法：在第一次训练中，将大模型的初始权重预训练为适应问题类型A和问题类型B的数据。然后，在第二次微调中，仅使用问题类型B的数据对模型进行微调。这样，模型的初步训练将更加专注于问题类型A，第二次微调则更专注于问题类型B。

调整学习率：在第二次微调期间，可以尝试降低学习率。较低的学习率可以减小模型对新数据的调整程度，从而限制对问题类型A回答效果的改变。

需要注意的是，即使采取了上述方法，模型在第二次微调后问题类型A的回答结果可能仍然会发生变化。这是因为模型会尽力适应新的训练数据，并可能对整体表现进行优化。因此，为了保持问题类型A的回答效果，建议在第二次微调后进行评估和调试，并进行必要的微调和修改。
``` 

尝试: 

  - 用lora进行第一次微调 (数据为问题类型A), 导出成模型, 再用lora进行第二次微调 (数据为问题类型B). 会导致第一次中训练的问题类型A偏离 (不论是否使用resume_lora_training)
  - 用lora进行第一次微调 (数据为问题类型A), 导出成模型, 再用freeze进行第二次微调 (数据为问题类型B). 会导致第一次中训练的问题类型A偏离
