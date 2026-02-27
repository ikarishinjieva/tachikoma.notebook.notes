---
title: 20230901 - chatglm 微调, 对sql进行规则审核 - 使用randgen生成训练SQL
confluence_page_id: 2589097
created_at: 2023-09-01T14:54:07+00:00
updated_at: 2023-09-12T02:07:34+00:00
---

使用randgen生成训练SQL, 参考[20230830 - 随机SQL生成], 注意: 

  - 目前仅对于一个 "恒等式" 规则进行训练
  - 需要提高epoch, 将loss降到0.001+, 目前epoch = 6, 使用4轮训练
  - 需要生成 最够多的"恒等式" SQL

效果: 

对于数字匹配, 大部分正确, 少量不正确可以通过训练数据进行修正

```
问：sql='''SELECT dddd as cccc FROM a WHERE 1 = 6''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 1 = 6''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

---

问：sql='''SELECT dddd as cccc FROM a WHERE 6 = 6''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

---

问：sql='''SELECT dddd as cccc FROM a WHERE 12345 = 13345''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 13345''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

---

问：sql='''SELECT dddd as cccc FROM a WHERE 12345 = 1345''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 1345''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
 
---
 
问：sql='''SELECT dddd as cccc FROM a WHERE -2 > -1''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -2 > -1''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
``` 

对文字的判断错误率高: 

```
问：sql='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc'''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

---

问：sql='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

---

问：sql='''SELECT dddd as cccc FROM a WHERE 'abceffff' = 'abcd'''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abceffff' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
``` 

对以下用例输出不稳定: 

```
问：sql='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

---

问：sql='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
 
``` 

增加生成数据的反例对比: 

```
not_degenerate_where_item:
    tinyint = tinyint |
    tinyint > tinyint |
    char = char |
	char NOT LIKE char |
    string_16 = string_16 | 
    string_16 NOT LIKE string_16;
``` 

效果: 以上问题大部分消除, 有少量 反例 用例不稳定: 

```
问：sql='''SELECT dddd as cccc FROM a WHERE 12345 = 12345''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[ 不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是 是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

-

问：sql='''SELECT dddd as cccc FROM a WHERE 12345 = 12345''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[ 不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否 否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

---

问：sql='''SELECT dddd as cccc FROM a WHERE -1 > -1''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是 是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

-

问：sql='''SELECT dddd as cccc FROM a WHERE -1 > -1''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否 否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
``` 

错误输出: 

```
问：sql='''SELECT count(*) FROM head WHERE 'age'  = 'age' for update''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

答：
子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age' for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
``` 

增加训练数据: 

  - 888个训练数据
  - "恒等条件"的数据为333个
  - 正向和反向数据分布: 
    - select 语句的where条件[不能使用]恒等条件
              positive cases = 555
              negative cases = 333
          select 语句的where条件[必须使用]恒等条件
              positive cases = 127
              negative cases = 761

使用eval测试数据集, 分析结果, 错误的结果都是 "[必须使用]"的案例, 猜测是因为"[必须使用]"的positive case数量不够: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
**Line 18:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否 否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

**Line 21:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

**Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345'''
结论: 否否否否否否否否否否

**Line 24:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是 是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否 否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

**Line 25:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''检查每一个Select语句, {AST}直接 使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否 否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

继续增加训练数据: 

  - 2336 / 4429  
degenerate_count=873

  - select 语句的where条件[不能使用]恒等条件  
positive cases = 1463  
negative cases = 873

  - select 语句的where条件[必须使用]恒等条件  
positive cases = 326  
negative cases = 2010

测试数据仅一例不通过: 

```
Line 14:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否 否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
``` 

降低epoch=4, 4轮, 评估结果: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
* Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500''' 否否否否否否否否否否

* Line 10:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update'''
子问题: sql[2]='''SELECT count(*) FROM head WHERE 'age'  != 'age'  for update'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 15:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500'''
子问题: sql[2]='''SELECT * FROM student WHERE GPA > 3.0'''
结论: 否否否否否否否否否否

* Line 21:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update'''
子问题: sql[2]='''SELECT count(*) FROM head WHERE 'age'  != 'age'  for update'''
结论: 否否否否否否否否否否

Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是 是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

对v2的评估结果, 会有更大偏差

# 尝试去除辅助数据 (SQL -> AST等数据), 只使用audits数据集

评估结果, 偏差增大: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
** Line 4:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 5:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500'''
 whereClause:'''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 7:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 8:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 9:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是 是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 10:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 11:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 12:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 13:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 14:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是 是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 15:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500'''
子问题: is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 16:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 17:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 18:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 19:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 22:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否 否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 24:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 27:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否 否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

# 尝试只使用audits+ast数据集

训练顺序: audits + ast + audits, 每个epoch=4

效果: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
** Line 5:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 '''''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 7:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 8:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a'''
子问题: is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 9:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc'''''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 11:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 12:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 13:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 14:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 '''''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 15:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是 是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500'''
子问题: is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 16:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 17:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

Line 18:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 19:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a'''''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

Line 22:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc'''''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 24:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

调整顺序: ast + audits + ast + audits, 每个epoch=4

```
Line 1:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 4:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''不包括'''不包含'''不包含''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''' ''''''''''

** Line 5:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否 否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否 否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

* Line 7:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', ast_rule='''包括但是不包含'''‘‘’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’'’'’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’’'’'’’'’'’’’'’'’'’'’'’’'’'’’’'’''''’’’’’’’’’’’’’’’’’’’’’’''''’’’’’’’’’’’’’’’’’''''’’’’’’’’’’’’’’''''’’’’''''’’’’''''’’''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''═════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════

** Line 8:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a''''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 9:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

Line 10:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', ast_rule='''检查每一个SelectStmt, { Ast }  直接使用的WhereClause { Ast }  inclusive?  else { Ast }  inclusive?  else { Ast }  else { Fail }''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 11:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 12:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 13:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id = T2.station_id WHERE 7 = 7 ''', ast_rule='''检查每一个Select语句, {AST}直 接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 14:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', ast_rule='''检查每一个SelectStmt, { Ast }直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 17:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 18:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 19:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a''''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 21:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', ast_rule='''检查每一个SelectStmt, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 22:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 24:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 25:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', ast_rule='''否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 27:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL 不满足规则''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

  

# 尝试只使用audits+rule+match数据集

训练顺序: audits + explains_and_rules + audits, 每个epoch=4

偏离更大: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
** Line 5:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 7:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 8:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a'''
结论: 满足规则

子问题: sql[2]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a'''
结论: 满足规则

结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 9:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 10:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 11:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 12:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 13:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 14:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 15:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是 是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 16:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 17:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

* Line 18:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1'''
结论: 满足规则
结论: 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否,

* Line 19:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a'''
结论: 满足规则
结论: 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否, 否,

* Line 22:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc'''
结论: 满足规则
结论: 否, 否则, 子问题不满足规则

** Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 24:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 27:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

调整训练顺序: explains_and_rules + audits + explains_and_rules + audits, 每个epoch=4

效果很差: 

```
** Line 5:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 7:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 8:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 9:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 10:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 11:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 12:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 13:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 14:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 15:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 16:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 17:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 18:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 19:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 20:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 21:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 22:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 24:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

# 尝试使用 all_except_audits + audit + all_except_audits + audit, 4轮训练, epoch =4

效果: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
* Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500''' 否否否否否否否否否否

* Line 10:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update'''
子问题: sql[2]='''SELECT count(*) FROM head WHERE 'age'  != 'age'  for update'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 15:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500'''
子问题: sql[2]='''SELECT * FROM student WHERE GPA > 3.0'''
结论: 否否否否否否否否否否

* Line 21:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update'''
子问题: sql[2]='''SELECT count(*) FROM head WHERE 'age'  != 'age'  for update'''
结论: 否否否否否否否否否否

Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 '''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

# 尝试不使用ast_rule

将所有使用到ast_rule的地方, 换成sql_rule

所有结果均是错误: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
* Line 1:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 异步检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则; 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则,  otherwise, 否则, 否则, 否则, 否则, 否则, 否则,

* Line 2:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12346''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 异构体检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则; 否则, SQL不满足规则;

* Line 3:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = unit)''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是 是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

* Line 4:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 检查每一个Select语句, {AST} 直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则;

* Line 5:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则;

* Line 6:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则。

* Line 7:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则;

* Line 8:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

* Line 9:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

* Line 10:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

* Line 11:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 异步checker检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则;

* Line 12:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则;

* Line 13:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 检查每一个Select语句, {AST}直接使用的WhereClause中是否[不包括]恒等条件; 否则, SQL不满足规则;

* Line 14:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则;

* Line 15:
        label: 子问题: sql[1]='''SELECT Official_Name FROM city WHERE Population  = Population OR 500 = 500 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 异步检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则; 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则,

* Line 16:
        label: 子问题: sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

* Line 17:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则。

* Line 18:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE -1 > -1 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则;

** Line 19:
        label: 子问题: sql[1]='''SELECT * FROM orders WHERE order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 异构体包括';' `ast(sql)`;
不满足规则;

* Line 20:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'abcd'  = 'abcd'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则'''

从AST规则检查开始, 检查每一个Select语句, {AST} 直接使用的WhereClause中是否[包括] 恒等条件; 否则, SQL不满足规则。
否则, SQL不满足规则。

* Line 21:
        label: 子问题: sql[1]='''SELECT count(*) FROM head WHERE 'age'  = 'age'  for update''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 异步检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则; 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则, 否则,

* Line 22:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abc' = 'abc' ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

* Line 23:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12345 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则;

* Line 24:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 6 = 6 ''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 否则, SQL不满足规则''', is_ast_match_ast_rule(ast(sql), ast_rule)=?

* Line 25:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE a = 1 and b > 2''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 异步检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否; 否则, 否;

* Line 26:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 12345 = 12346''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则; 否则, SQL不满足规则;

* Line 27:
        label: 子问题: sql[1]='''SELECT dddd as cccc FROM a WHERE 'abce' = 'abcd'''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则''`

由于 `ast_rule` 检查每一个 Select语句, {AST} 直接使用的 WhereClause 中是否 [包括] 恒等条件; 否则, SQL 不满足规则`, 可知答案为：是，是是是是是是是是是是

* Line 28:
        label: 子问题: sql[1]='''SELECT product_name, unit_price FROM products WHERE (category_id = 2) OR (unit_price > 100 AND 'blabla' = unit)''', is_ast_match_ast_rule(ast(sql[1]), ast_rule)='''否否否 否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 否则, SQL不满足规则。

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

# 增加一组规则

用例变多, 将epoch从4.0调为3.0

训练数据使用采样, 采样比: 

```
select语句[必须使用]where条件
    positive cases = 193
    negative cases = 500
select语句[不能使用]where条件
    positive cases = 500
    negative cases = 500
 
 
 
select 语句的where条件[不能使用]恒等条件
    positive cases = 1000
    negative cases = 568
select 语句的where条件[必须使用]恒等条件
    positive cases = 217
    negative cases = 1000
``` 

结果: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning# python ./analyze.py
** Line 1:
        label: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

Line 2:
        label: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE  7  =  7 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE  7  =  7 ''''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 3:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 4:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 5:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12346''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12346''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 6:
        label: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = unit)''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = unit)''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 7:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abce' = 'abcd'''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abce' = 'abcd'''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 8:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 9:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 10:
        label: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 11:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 12:
        label: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''是是是是是是是 是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''否否否否否否否 否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 13:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 14:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  a = 1 and b > 2''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  a = 1 and b > 2''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

* Line 16:
        label: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
子问题: sub_sql[2]='''select c from d group by u''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

* Line 24:
        label: 子问题: sub_sql[1]='''select * from a as a''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[2]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[3]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[4]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[5]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[6]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[7]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[8]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[9]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[10]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[11]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[12]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[13]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[14]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[15]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[16]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[17]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[18]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[19]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[20]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[21]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[22]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[23]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[24]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[25]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[26]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[27]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[28]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[29]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[30]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[31]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[32]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[33]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[34]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[35]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[36]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[37]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[38]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[39]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[40]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[41]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[42]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[43]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[44]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[45]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[46]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[47]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[48]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[49]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[50]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[51]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[52]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[53]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[54]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[55]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[56]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[57]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[58]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[59]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[60]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[61]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[62]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[63]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[64]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[65]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[66]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[67]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[68]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[69]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[70]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[71]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[72]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[73]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[74]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[75]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[76]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[77]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[78]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[79]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[80]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[81]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[82]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[83]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[84]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[85]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[86]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[87]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[88]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[89]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[90]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[91]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[92]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[93]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[94]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[95]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[96]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[97]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[98]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[99]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[100]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[101]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[102]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[103]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[104]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[105]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[106]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[107]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[108]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[109]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[110]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[111]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[112]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[113]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[114]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[115]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[116]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[117]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[118]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[119]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[120]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[121]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[122]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[123]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[124]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[125]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[126]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[127]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[128]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[129]='''select * from a as a''', top_sql_num=2;  子问题: sub_sql[130]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[131]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[132]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[133]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[134]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[135]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[136]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[137]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[138]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[139]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[140]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[141]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[142]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[143]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[144]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[145]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[146]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[147]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[148]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[149]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[150]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[151]='''select * from a as a''', top_sql_num=2; 子问 题: sub_sql[152]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[153]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[154]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[155]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[156]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[157]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[158]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[159]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[160]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[161]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[162]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[163]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[164]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[165]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[166]='''select * from a as a''', top_sql_num=2; 子问题: sub_sql[167]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[168]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[169]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[170]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[171]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[172]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[173]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[174]='''select * * from a as a''', top_sql_num=2; 子问题: sub_sql[175]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[176]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[177]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[178]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[179]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[180]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[181]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[182]='''select * * * from a as a''', top_sql_num=2; 子问题: sub_sql[183]='''select * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[184]='''select * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[185]='''select * * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[186]='''select * * * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[187]='''select * * * * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[188]='''select * * * * * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[189]='''select * * * * * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[190]='''select * * * * * * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[191]='''select * * * * * * * * * from a as a''', top_sql_num=2; 子问题: sub_sql[192]='''select * * * * * * * * * * select * * * * * * * * * * * select * select * select * * Select * Select * Select * Select * Select * Select * Select * Select * Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select Select

* Line 25:
        label: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
子问题: sub_sql[2]='''select c from d group by u''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 31:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  a = 1 and b > 2''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  a = 1 and b > 2''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 32:
        label: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''否否否否否否否 否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''是是是是是是是 是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 33:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

Line 34:
        label: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE  7  =  7 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE  7  =  7 ''''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 35:
        label: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 36:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 37:
        label: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = unit)''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = unit)''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 38:
        label: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 39:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abce' = 'abcd'''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abce' = 'abcd'''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 40:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 41:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 42:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12346''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12346''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 43:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 44:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

* Line 53:
        label: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
子问题: sub_sql[2]='''select c from d group by u''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 57:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 58:
        label: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE  7  =  7 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE  7  =  7 ''''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 59:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 60:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 61:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 62:
        label: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 63:
        label: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 64:
        label: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''否否否否否否否 否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''是是是是是是是 是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 65:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 66:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则

** Line 67:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  12345 = 12345 ''''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 68:
        label: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT product_name, unit_price FROM products WHERE  (category_id = 2) OR (unit_price > 100 AND 'blabla' = 'blabla') ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 69:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'age'  = 'age'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 70:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  -1 > -1 ''''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 71:
        label: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE  7  =  7 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT DISTINCT T1.name FROM station AS T1 JOIN status AS T2 ON T1.id  =  T2.station_id WHERE 7  =  7 ''''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 72:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  6 = 6 ''''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 73:
        label: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''是是是是是是是 是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT * FROM orders WHERE  order_date <= CURRENT_DATE() AND 'a' = 'a' ''', is_sub_sql_match_ast_rule='''否否否否否否否 否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 74:
        label: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT Official_Name FROM city WHERE  Population  = Population OR 500 = 500 ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 75:
        label: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT dddd as cccc FROM a WHERE  'abc' = 'abc' ''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 76:
        label: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''是是是是是是是是是是'''
结论: 所有检查均满足规则, 结论为该SQL满足规则
        predi: 子问题: sub_sql[1]='''SELECT count(*) FROM head WHERE  'abcd'  = 'abcd'  for update''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

Line 81:
        label: 子问题: sub_sql[1]='''SELECT * FROM (   SELECT id, name   FROM products   ORDER BY name DESC ) t ORDER BY id''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
子问题: sub_sql[2]='''SELECT id, name FROM products ORDER BY name DESC''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''SELECT * FROM (   SELECT id, name   FROM products   ORDER BY name DESC ) t ORDER BY id''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
子问题: sub_sql[2]='''SELECT id, name   FROM products''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

** Line 84:
        label: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
子问题: sub_sql[2]='''select c from d group by u''', is_sub_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则
        predi: 子问题: sub_sql[1]='''select 1 from a join (select c from d group by u) b group by e''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_is_sql_match_ast_rule='''否否否否否否否否否否'''
子问题: sub_sql[2]='''select c from d''', ast_rule='''检查每一个Select语句, {AST}直接使用的WhereClause中是否[包括]恒等条件; 否则, SQL不满足规则''', is_is_sql_match_ast_rule='''否否否否否否否否否否'''
结论: 有步骤不满足规则, 所以该SQL不满足规则

(chatglm_etuning) root@gpu198:/home/user01/sql_audit/ChatGLM-Efficient-Tuning#
``` 

其中: 

  - 前44个为 是否有where语句的规则, 结论失败26个
  - 后44个为恒等条件的规则, 结论失败21个

epoch调小 以及 训练数据变少, 导致恒等条件的失败率升高. 即使这样, 目前在双卡4090上, 依然需要6小时.
