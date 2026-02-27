---
title: 20230620 - 大模型chatglm微调, 样本对模型的影响 (3)
confluence_page_id: 2588700
created_at: 2023-06-20T12:43:22+00:00
updated_at: 2023-06-24T09:27:39+00:00
---

# 现象

改变规则的限制数, 结论会出现错误

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须带limit,且限制数不得超过1001>>

ChatGLM-6B: 结论： 不符合
分析： 该SELECT语句没有使用`LIMIT`关键字， 不符合规则中"select 语句必须带limit"的要求
``` 

调整训练数据, 增加限制数的多样性记录(增加了第一条): 

```
  [{
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过2000>>",
    "input": "",
    "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
  },{
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n"
  },
  {
    "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
    "input": "",
    "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
  },
  {
    "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
    "input": "",
    "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
  }]
``` 

效果: 

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须带limit,且限制数不得超过1001>>

ChatGLM-6B: 结论： 符合
分析： 该SELECT语句使用了`LIMIT 1000`子句， 符合规则中"select 语句必须带limit"的要求， 限制数为1000未超过了阈值1001, 符合规则中"限制数不得超过1001"的要求
``` 

# 现象2

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1001;>> 是否符合我们定义的规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 符合
分析： 该SELECT语句使用了`LIMIT 1001`子句， 符合规则中"select 语句必须带limit"的要求， 限制数为1001未超过了阈值1000, 符合规则中"限制数不得超过1000"的要求
``` 

阈值的比较 不正确

分析:

```
User: 1001是否小于等于1000

ChatGLM-6B: 是的，1001 小于等于 1000。
``` 

增加数字比较的训练数据, 使用脚本生成测试数据 10000 组, 调整训练参数: 

```
CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://127.0.0.1:7890 python src/train_sft.py     --do_train     --dataset sql_audit     --finetuning_type lora     --output_dir path_to_sft_checkpoint.sql_audit     --per_device_train_batch_size 5     --gradient_accumulation_steps 4     --lr_scheduler_type cosine     --logging_steps 10     --save_steps 1000     --learning_rate 5e-5     --num_train_epochs 2     --fp16
``` 

batch=5, epoch=2, 可比较常用数字的大小. 

将训练的模型(可以比较数字大小的模型)进行导出

之后的微调基于这个模型: 

```
CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://127.0.0.1:7890 python src/train_sft.py     --do_train  --model_name_or_path model.number_comparision   --dataset sql_audit     --finetuning_type lora     --output_dir path_to_sft_checkpoint.sql_audit     --per_device_train_batch_size 1     --gradient_accumulation_steps 4     --lr_scheduler_type cosine     --logging_steps 10     --save_steps 1000     --learning_rate 5e-5     --num_train_epochs 100     --fp16
``` 

效果: 

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1001;>> 是否符合我们定义的规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 结论：符合
分析：该SELECT语句使用了`LIMIT 1001`子句，符合规则中"select 语句必须带limit"的要求，限制数为1001，超过了阈值1000，没有使用`&`运算符，也没有使用关键字`AS`，不符合规则中"限制数不得超过1000"的要求
高亮：该SELECT语句使用了`LIMIT 1001`子句，符合规则中"select 语句必须带limit"的要求
<b>结论：</b> 符合
高亮：该SELECT语句使用了`LIMIT 1001`子句
<b>分析：</b> 符合规则中"select 语句必须带limit"的要求，限制数为1001，超过了阈值1000，没有使用`&`运算符，不符合规则中"限制数不得超过1000"的要求
高亮：该SELECT语句没有使用关键字`AS`，不符合规则中"限制数不得超过1000"的要求
结论：不符合
高亮：该SELECT语句使用了`LIMIT 1001`子句
<b>分析：</b> 不符合规则中"限制数不得超过1000"的要求
高亮：该SELECT语句没有使用关键字`AS`，使用了`*`作为关键字，不符合规则中"限制数不得超过1000"的要求
结论：不符合
高亮：该SELECT语句使用了`LIMIT 1001`子句
<b>分析：</b> 不符合规则中"限制数不得超过1000"的要求
高亮：该SELECT语句使用了`&`运算符，没有使用关键字`AS`，不符合规则中"限制数不得超过1000"的要求
结论：不符合
高亮：该SELECT语句使用了`LIMIT 1001`子句
<b>分析：</b> 不符合规则中"限制数不得超过1000"的要求
高亮：该SELECT语句使用了`&`运算符，使用了关键字`AS`，使用了`*`作为关键字，不符合规则中"限制数不得超过1000"的要求
结论：不符合
高亮：该SELECT语句使用了`LIMIT 1001`子句
总结：该SELECT语句不符合我们定义的规则 <<select 语句必须带limit，且限制数不得超过1000>>
``` 

阈值比较的结果已经正确. 但:

  1. 分析中的"不符合" -> 结论中的"符合" 这个逻辑链是断的
  2. 分析中多产生了 "没有使用`&`运算符，也没有使用关键字`AS`，" 这个逻辑
  3. "高亮"部分会生成错误

修订数据集 (增加第一条数据), 增加 分析中的"不符合" -> 结论中的"符合" 的逻辑: 

```
[{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1010</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过2000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},
{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n"
},
{
  "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
  "input": "",
  "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
},
{
  "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
  "input": "",
  "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
}]
``` 

测试效果: 

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1001;>> 是否符合我们定义的规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 结论：不符合
分析：该SELECT语句使用了`LIMIT 1001`子句，符合规则中"select 语句必须带limit"的要求，限制数为1001，超过了阈值1000，不符合规则中"限制数不得超过1000"的要求
``` 

# 现象3

limit和limit子句是有差别的. 之后统一使用"limit子句"

```
User: <<select * from a where a in (select b from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须使用limit,且限制数不得超过1000>>

ChatGLM-6B: 根据您定义的规则，该SELECT语句不符合限制数不得超过1000的要求，未使用`limit`语句。

该SELECT语句属于`where`子句，子句内部使用了`in`运算符，查询了`c`表中的`b`字段与`a`字段是否相等。该语句使用了LIMIT 1999次预查询，获得了`a`字段所有值为`b`的记录。

因此，该SELECT语句获得了超过限制数1000的记录，不符合您定义的规则`select 语句必须使用limit，且限制数不得超过1000`。
 
------
 
User: <<select * from a where a in (select b from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SELECT语句使用了`limit 1999`子句， 符合规则中"select 语句必须使用limit子句"的要求， 限制数为1999超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求
高亮： `select * from a where a in (select b from c) limit 1999`
``` 

调整测试集, 统一"LIMIT子句": 

```
[{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1010</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},
{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n"
},
{
  "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
  "input": "",
  "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
},
{
  "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
  "input": "",
  "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
}]
``` 

效果: 

```
User: <<select * from a where a in (select b from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论：不符合
分析：该SELECT语句使用了`limit 1999`子句，不符合规则中"select 语句必须显式使用limit，且限制数不得超过1000"的要求
高亮：该SELECT语句的`limit 1999`子句超过了规则中"限制数不得超过1000"的限制，不符合要求
建议：该SELECT语句的`limit 1999`子句应该改为小于或等于1000的数值，符合规则要求
高亮：该SELECT语句的`limit 1999`子句使用了`*`，不符合规则中"select 语句必须显式使用limit，且限制数不得超过1000"的要求
建议：该SELECT语句应该去掉`*`，符合规则要求
高亮：该SELECT语句的WHERE子句中使用了`in (select b from c)`，不符合规则中"select 语句必须显式使用limit，且限制数不得超过1000"的要求
建议：该SELECT语句应该去掉`in (select b from c)`，符合规则要求
高亮：该SELECT语句的WHERE子句中没有显式使用`limit`，不符合规则中"select 语句必须显式使用limit，且限制数不得超过1000"的要求
建议：该SELECT语句应该加上`limit 1999`，符合规则要求

---
User: <<select * from a where a in (select b from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SELECT语句没有显式使用`limit`, 不符合规则中"select 语句必须显式使用limit子句"的要求

---

User: <<select * from a where a = 1 limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用LIMIT子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SELECT语句使用了`limit 1999`子句， 符合规则中"select 语句必须显式使用LIMIT子句"的要求， 限制数为1999, 超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求

---

User: <<select * from a where a in (1) limit 999>> 是否符合我们定义的规则 <<select 语句必须显式使用LIMIT子句,且限制数不得超过1000>>

ChatGLM-6B: 结论：不符合
分析：该SELECT语句使用了`limit 999`子句，符合规则中"select 语句必须显式使用LIMIT子句"的要求，限制数为999，未超过了阈值1000，符合规则中"限制数不得超过1000"的要求
高亮：该SELECT语句使用了`limit 999`子句
结论：不符合
分析：该SELECT语句使用了`limit 999`子句，限制数为999，符合规则中"select 语句必须显式使用LIMIT子句"的要求，未超过了阈值1000，符合规则中"限制数不得超过1000"的要求
高亮：该SELECT语句使用了`limit 999`子句
``` 

# 现象4

需增加对子查询的处理: 

```
User: <<select * from a where a in (select b from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SELECT语句没有显式使用`limit`, 不符合规则中"select 语句必须显式使用limit子句"的要求
``` 

修订训练数据: 

```
[{
  "instruction": "<<select * from a where a in (select b,a from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 1. 该SQL包含子查询`select b,a from c`, 这个子查询是SELECT语句但没有使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 1999`, 符合规则中
\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<select * from a where a in (select b from d limit 100) limit 200>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 1. 该SQL包含子查询`select b from d limit 100`, 这个子查询是是SELECT语句并使用了`LIMIT 100`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为100不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 200`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为200不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1010</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},
{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n"
},
{
  "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
  "input": "",
  "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
},
{
  "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
  "input": "",
  "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
}]
``` 

效果: 

```
User: <<select * from a where a in (select b from c group by b limit 10) limit 99>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select b from c group by b limit 10`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 99`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为99超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求

(子查询是否使用LIMIT的判断错误)

---

User: <<select * from a where a in (select 1 from c where d = 1) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SQL语句使用了`limit 987`子句， 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987未超过了阈值1000, 符合规则中"限制数不得超过1000"的要求
高亮： `select * from a where a in (select 1 from c where d = 1)`
该SQL语句中的`select * from a`是select语句， 没有使用`limit 987`, 不符合规则中"select 语句必须显式使用limit子句"的要求
低亮： `limit 987`
该SQL语句中的`limit 987`是limit子句， 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987未超过了阈值1000, 符合规则中"限制数不得超过1000"的要求
总结： 该SQL语句符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987未超过了阈值1000, 符合规则中"限制数不得超过1000"的要求

(无法识别子查询)

---

User: <<select * from a where a in (select p from c where d = 1) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select p from c where d = 1`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求

(可以识别子查询, 对阈值大小判断错误)

---

User: <<select * from a where a in (select p from c where d = (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SQL语句使用了`limit 987`子句， 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987未超过了阈值1000, 符合规则中"限制数不得超过1000"的要求

(无法识别子查询)

---

User: <<select * from a where a in (select p from c where d in (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select * from c where a in (select p from c where d in (select t from e))`, 这个子查询是包含查询`a`在`c`和`e`中的所有`p`和`d`的值， 也就是包含了查询`a`在`c`和`e`中所有可能匹配的`p`和`d`的值， 这个子查询没有使用`limit 987`, 不符合规则中"select 语句必须显式使用limit子句"的要求
 2. 该SQL的子查询`select * from c where a in (select p from c where d in (select t from e))`是包含查询`a`在`c`和`e`中所有可能匹配的`p`和`d`的值， 也就是包含了查询`a`在`c`和`e`中所有可能匹配的`p`和`d`的值中的`p`和`d`的值， 只包含了`c`和`e`中的两个查询， 没有包含第三个查询中的`p`和`d`, 不符合规则中"select 语句必须显式使用limit子句"的要求
 3. 该SQL的限制数是987, 超过了规则中"限制数不得超过1000"的要求， 不符合规则中"select 语句必须显式使用limit子句"的要求

 (子查询识别错误)
``` 

针对子查询识别错误, 增加训练数据: 

```
[{
  "instruction": "<<select * from a where a in (select b,a from c limit 1) limit 1>> 包含几个子查询",
  "input": "",
  "output": "包含1个子查询: \n 1. `select b,a from c limit 1` 是子查询1"
},{
  "instruction": "<<select * from a where a in (select b,a from c where d in (select 1 from dual) limit 1) order by a>> 包含几个子查询",
  "input": "",
  "output": "包含2个子查询: \n 1. `select b,a from c where d in (select 1 from dual) limit 1` 是子查询1 \n 2. `select 1 from dual` 是子查询2"
},{
  "instruction": "<<select * from a where a in (select b,a from c) limit 10>> 包含几个子查询",
  "input": "",
  "output": "包含1个子查询: \n 1. `select b,a from c` 是子查询1"
},{
  "instruction": "<<select * from a where a in (select b,a from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 1. 该SQL包含子查询`select b,a from c`, 这个子查询是SELECT语句但没有使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<select * from a where a in (select b from d limit 100) limit 200>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 1. 该SQL包含子查询`select b from d limit 100`, 这个子查询是是SELECT语句并使用了`LIMIT 100`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为100不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 200`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为200不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1010</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},
{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n"
},
{
  "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
  "input": "",
  "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
},
{
  "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
  "input": "",
  "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
}]
``` 

效果: 

```
User: <<select * from a where a in (select b from c group by b limit 10) limit 99>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select b from c group by b limit 10`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求.
 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 10`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为99超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求.

(子查询是否使用LIMIT的判断错误)

 ---

 User: <<select * from a where a in (select 1 from c where d = 1) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select 1 from c where d = 1`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求.
 2. 该SQL的最外层查询是SELECT语句并使用了`in`子句， 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求.

(可以正确识别子查询)

 ---

 User: <<select * from a where a in (select p from c where d = 1) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select p from c where d = 1`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 2. 该SQL的最外层查询是SELECT语句并使用了`p`关键字， 属于规则中"select 语句必须显式使用limit子句"的要求， 符合规则中"限制数不得超过1000"的要求， LIMIT子句限制查询结果数为987, 符合规则中的要求

(可以识别子查询, 对阈值大小判断错误)

 ---

 User: <<select * from a where a in (select p from c where d = (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论：不符合
分析：
该SQL语句使用了`in`子句，属于select语句，但没有使用`limit`子句，不符合规则中"select 语句必须显式使用limit子句"的要求，限制数为987超过了阈值1000，不符合规则中"限制数不得超过1000"的要求
高亮：
 * 该SQL语句中的`select p from c where d = (select t from e)`属于select语句，但没有使用`limit`，不符合规则中"select 语句必须显式使用limit子句"的要求
 * 该SQL语句中的`select t from e`属于select语句，但没有使用`limit`，不符合规则中"select 语句必须显式使用limit子句"的要求
 * 该SQL语句中的`a in (select p from c where d = (select t from e))`属于in子句，符合规则中"select 语句必须显式使用limit子句"的要求，限制数为987符合规则中"限制数不得超过1000"的要求
低亮：
 * 该SQL语句中使用了`limit 987`，属于select语句中使用limit子句，符合规则中"select 语句必须显式使用limit子句"的要求，限制数为987符合规则中"限制数不得超过1000"的要求

(子句拆解有问题)
 ---

 User: <<select * from a where a in (select p from c where d in (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论：不符合
分析：
该SQL语句使用了`in`子句，属于select语句，但没有使用`limit`子句，不符合规则中"select 语句必须显式使用limit子句"的要求，限制数超过了阈值1000，不符合规则中"限制数不得超过1000"的要求
高亮：a in (select p from c where d in (select t from e))
属于select语句，a是c的子查询，d是e的子查询，不符合规则中"select 语句必须显式使用limit子句"的要求
高亮：LIMIT 987
没有使用select语句，不符合规则中"限制数不得超过1000"的要求
总结：该SQL语句符合规则中"select 语句必须显式使用limit子句"的要求，限制数为987，属于select语句，是c的子查询，符合规则中"限制数不得超过1000"的要求，属于未使用select语句的子查询，不符合规则中"select 语句必须显式使用limit子句"的要求

(子句拆解有问题)
``` 

增加训练epoch=200后, 效果改善: 

```
User: <<select * from a where a in (select p from c where d = (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select p from c where d = (select t from e)`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 2. 该SQL的最外层查询是SELECT语句并使用了子查询`select p from c where d = (select t from e)`, 这个子查询是SELECT语句并使用了LIMIT子句， 符合规则中"select 语句必须显式使用limit子句"的要求
 
---
 
User: <<select * from a where a in (select p from c where d in (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select p from c where d in (select t from e)`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 2. 该SQL的最外层查询是SELECT语句并使用了子查询`select p from c where d in (select t from e)`, 子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 
(只能拆解一层子查询)
``` 

调整训练数据: 

  - 明确 "LIMIT子句"
  - 将子查询从select 1 from dual改成实际表

```
[{
  "instruction": "<<select * from a where a in (select b,a from c limit 1) limit 1>> 包含几个子查询",
  "input": "",
  "output": "包含1个子查询: \n 1. `select b,a from c limit 1` 是子查询1"
},{
  "instruction": "<<select * from a where a in (select b,a from c where d in (select e from f) limit 1) order by a>> 包含几个子查询",
  "input": "",
  "output": "包含2个子查询: \n 1. `select b,a from c where d in (select e from f) limit 1` 是子查询1 \n 2. `select e from f` 是子查询2"
},{
  "instruction": "<<select * from a where a in (select b,a from c) limit 10>> 包含几个子查询",
  "input": "",
  "output": "包含1个子查询: \n 1. `select b,a from c` 是子查询1"
},{
  "instruction": "<<select * from a where a in (select b,a from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 1. 该SQL包含子查询`select b,a from c`, 这个子查询是SELECT语句但没有使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n 2. 该SQL的最外层查询是SELECT语句并使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<select * from a where a in (select b from d limit 100) limit 200>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 1. 该SQL包含子查询`select b from d limit 100`, 这个子查询是是SELECT语句并使用了LIMIT子句`LIMIT 100`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为100不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n 2. 该SQL的最外层查询是SELECT语句并使用了`LIMIT 200`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为200不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1010</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},
{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n"
},
{
  "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
  "input": "",
  "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
},
{
  "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
  "input": "",
  "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
}]

``` 

效果: 

```
User: <<select * from a where a in (select p from c where d = (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select t from e`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 2. 该SQL的最外层查询是SELECT语句并使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求

(只能认知到 最内层和最外层 的查询, 不能认知到中间层的查询)
  
---
  
User: <<select * from a where a in (select p from c where d in (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 1. 该SQL包含子查询`select t from e`, 这个子查询是包含查询`select t from e`的子查询， 没有使用`limit`子句， 不符合规则中"select语句必须显式使用limit子句"的要求
 2. 该SQL的最外层查询是包含查询`select t from e`, 最内层查询是包含查询`select p from c`, 这两个查询都是包含查询`select t from e`, 没有使用`limit`子句， 不符合规则中"select语句必须显式使用limit子句"的要求
  
(能拆解两层查询, 但分析格式需优化)

``` 

调整训练数据, 将分析步骤中的子查询步骤进行细化: 

```
[{
  "instruction": "<<select * from a where a in (select b,a from c limit 1) limit 1>> 包含几个子查询",
  "input": "",
  "output": "包含1个子查询: \n 1. `select b,a from c limit 1` 是子查询1"
},{
  "instruction": "<<select * from a where a in (select b,a from c where d in (select e from f) limit 1) order by a>> 包含几个子查询",
  "input": "",
  "output": "包含2个子查询: \n 1. `select b,a from c where d in (select e from f) limit 1` 是子查询1 \n 2. `select e from f` 是子查询2"
},{
  "instruction": "<<select * from a where a in (select b,a from c) limit 10>> 包含几个子查询",
  "input": "",
  "output": "包含1个子查询: \n 1. `select b,a from c` 是子查询1"
},{
  "instruction": "<<select * from a where a in (select b,a from c where b in (select e from d)) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SQL包含2个子查询: \n子查询1: `select e from d`, 这个子查询是SELECT语句但没有使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n子查询2: `select b,a from c where b in (select e from d)`, 这个子查询是SELECT语句但没有使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n最外层查询: 是SELECT语句并使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<select * from a where a in (select b,a from c where b in (select e from d limit 876)) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SQL包含2个子查询: \n子查询1: `select e from d limit 876`, 这个子查询是SELECT语句并使用了LIMIT子句`LIMIT 876`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为876不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n子查询2: `select b,a from c where b in (select e from d limit 876)`, 这个子查询是SELECT语句但没有使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n最外层查询: 是SELECT语句并使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<select * from a where a in (select b,a from c) limit 1999>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SQL包含1个子查询: \n子查询1: `select b,a from c`, 这个子查询是SELECT语句但没有使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n最外层查询: 是SELECT语句并使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<select * from a where a in (select b from d limit 100) limit 200>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SQL包含1个子查询: \n子查询1: `select b from d limit 100`, 这个子查询是是SELECT语句并使用了LIMIT子句`LIMIT 100`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为100不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n最外层查询: 是SELECT语句并使用了`LIMIT 200`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为200不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1010</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 符合\n分析: 该SELECT语句使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n"
},
{
  "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
  "input": "",
  "output": "结论: 不符合\n分析: 该SELECT语句没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n"
},
{
  "instruction": "<<SELECT a FROM t>> 是否使用了LIMIT子句",
  "input": "",
  "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
},
{
  "instruction": "<<SELECT a FROM t limit 10>> 是否使用了LIMIT子句",
  "input": "",
  "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
}]

``` 

效果: 

```
User: <<select * from a where a in (select p from c where d in (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SQL包含2个子查询：
子查询1: `select p from c where d in (select t from e)`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
子查询2: `select t from e`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
最外层查询： 是SELECT语句并使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求
 
------
 
User: <<select * from a where a in (select p from c where d = (select t from e)) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SQL包含1个子查询：
子查询1: `select p from c where d = (select t from e)`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
最外层查询： 是SELECT语句并使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求

 
(需要扩展子查询的定义)
 
------
 
User: <<select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987>> 是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>

ChatGLM-6B: 结论： 不符合
分析： 该SQL包含2个子查询：
子查询1: `select t from e limit 2`, 这个子查询是SELECT语句并使用了LIMIT子句`LIMIT 2`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为2, 不超过阈值1000, 符合规则中"限制数不得超过1000"的要求
子查询2: `select p from c where d in (select t from e) limit 1`, 这个子查询是SELECT语句但没有使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
最外层查询： 是SELECT语句并使用了LIMIT子句`LIMIT 987`, 限制数为987, 符合规则中"限制数不得超过1000"的要求
 
(对子查询的边界识别有误)
 
 
``` 

调整训练数据, 将input和instruction分开: 

```
[
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987",
        "output": "包含2个子查询: \n - `select p from c where d = (select t from e) limit 2` 是子查询1 \n - `select t from e` 是子查询2"
    },{
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where b in (select c from d where e in (select f from (select h from g)) limit 5) order by b",
        "output": "包含3个子查询: \n - `select c from d where e in (select f from (select h from g)) limit 5` 是子查询1 \n - `select f from (select h from g)` 是子查询2 \n - `select h from g` 是子查询3"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c where d in (select e from f limit 3) limit 5) order by a",
        "output": "包含2个子查询: \n - `select b,a from c where d in (select e from f limit 3) limit 5` 是子查询1 \n - `select e from f limit 3` 是子查询2"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a from (select d.c from d where a in (select b,a from c) limit 10)",
        "output": "包含2个子查询: \n - `select d.c from d where a in (select b,a from c) limit 10` 是子查询1\n - `select b,a from c` 是子查询2"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c limit 1) limit 5",
        "output": "包含1个子查询: \n - `select b,a from c limit 1` 是子查询1"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c) limit 10",
        "output": "包含1个子查询: \n - `select b,a from c` 是子查询1"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c limit 7) limit 10",
        "output": "包含1个子查询: \n - `select b,a from c limit 7` 是子查询1"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b,a from c where b in (select e from d)) limit 1999",
        "output": "结论: 不符合\n分析: 该SQL包含2个子查询: \n - 子查询1: `select e from d`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n - 子查询2: `select b,a from c where b in (select e from d)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n - 主查询: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b,a from c where b in (select e from d limit 876)) limit 1999",
        "output": "结论: 不符合\n分析: 该SQL包含2个子查询: \n - 子查询1: `select e from d limit 876`, 这个子查询是SELECT语句并显式使用了LIMIT子句`LIMIT 876`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制 数为876不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n - 子查询2: `select b,a from c where b in (select e from d limit 876)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n - 主查询: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where b in (select d from c) limit 1999",
        "output": "结论: 不符合\n分析: 该SQL包含1个子查询: \n - 子查询1: `select d from c`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n - 主查询: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b from d limit 100) limit 200",
        "output": "结论: 符合\n分析: 该SQL包含1个子查询: \n - 子查询1: `select b from d limit 100`, 这个子查询是是SELECT语句并显式使用了LIMIT子句`LIMIT 100`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制 数为100不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n - 主查询: 是SELECT语句并使用了`LIMIT 200`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为200不超过阈值1000, 符合规则中\"限 制数不得超过1000\"的要求\n"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;",
        "output": "结论: 不符合\n分析: 主查询显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;",
        "output": "结论: 符合\n分析: 主查询显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;",
        "output": "结论: 符合\n分析: 主查询显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1",
        "output": "结论: 不符合\n分析: 主查询没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否使用了LIMIT子句",
        "input": "SELECT a FROM t",
        "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
    },
    {
        "instruction": "这个SQL是否使用了LIMIT子句",
        "input": "SELECT a FROM t limit 10",
        "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
    }
]

``` 

效果: 

即使我们将该SQL的子查询加入训练数据, 也没有改变对SQL的规则解析, 也就是说两种数据并没有被关联??

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>:select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987

ChatGLM-6B: 结论： 不符合
分析： 该SQL包含2个子查询：
 - 子查询1: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 - 子查询2: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 - 主查询： 是SELECT语句并显式使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求
所以该SQL限制数为987超过了规则中"限制数不得超过1000"的限制
``` 

增加训练epoch=100, 缩小学习率为 1e-5, 效果增强

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>:select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987

ChatGLM-6B: 结论： 不符合
分析： 主查询包含两个子查询：
- `select p from c where d in (select t from e) limit 2` 是子查询1, 是符合 `select 语句必须显式使用limit子句` 规则的， 也是主查询的一部分
- `select t from e` 是子查询2, 不是主查询， 不符合 `limit 987` 限制不得超过1000 规则
``` 

调整训练数据, 强调主查询概念: 

```
[
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987",
        "output": "包含2个子查询: \n - `select p from c where d = (select t from e) limit 2` 是子查询1 \n - `select t from e` 是子查询2"
    },{
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where b in (select c from d where e in (select f from (select h from g)) limit 5) order by b",
        "output": "包含3个子查询: \n - `select c from d where e in (select f from (select h from g)) limit 5` 是子查询1 \n - `select f from (select h from g)` 是子查询2 \n - `select h from g` 是子查询3"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c where d in (select e from f limit 3) limit 5) order by a",
        "output": "包含2个子查询: \n - `select b,a from c where d in (select e from f limit 3) limit 5` 是子查询1 \n - `select e from f limit 3` 是子查询2"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a from (select d.c from d where a in (select b,a from c) limit 10)",
        "output": "包含2个子查询: \n - `select d.c from d where a in (select b,a from c) limit 10` 是子查询1\n - `select b,a from c` 是子查询2"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c limit 1) limit 5",
        "output": "包含1个子查询: \n - `select b,a from c limit 1` 是子查询1"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c) limit 10",
        "output": "包含1个子查询: \n - `select b,a from c` 是子查询1"
    },
    {
        "instruction": "这个SQL包含几个子查询",
        "input": "select * from a where a in (select b,a from c limit 7) limit 10",
        "output": "包含1个子查询: \n - `select b,a from c limit 7` 是子查询1"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b,a from c where b in (select e from d)) limit 1999",
        "output": "结论: 不符合\n分析: - 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含2个子查询: \n \t- 子查询1: `select e from d`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n \t- 子查询2: `select b,a from c where b in (select e from d)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b,a from c where b in (select e from d limit 876)) limit 1999",
        "output": "结论: 不符合\n分析:  - 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含2个子查询: \n \t- 子查询1: `select e from d limit 876`, 这个子查询是SELECT语句并显式使用了LIMIT子句`LIMIT 876`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制 数为876不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n \t- 子查询2: `select b,a from c where b in (select e from d limit 876)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where b in (select d from c) limit 1999",
        "output": "结论: 不符合\n分析: - 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含1个子查询: \n \t- 子查询1: `select d from c`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b from d limit 100) limit 200",
        "output": "结论: 符合\n分析: - 主语句: 是SELECT语句并使用了`LIMIT 200`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为200不超过阈值1000, 符合规则中\"限 制数不得超过1000\"的要求\n - 主语句包含1个子查询: \n \t- 子查询1: `select b from d limit 100`, 这个子查询是是SELECT语句并显式使用了LIMIT子句`LIMIT 100`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制 数为100不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;",
        "output": "结论: 不符合\n分析: - 主语句: 显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;",
        "output": "结论: 符合\n分析: - 主语句: 显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;",
        "output": "结论: 符合\n分析: - 主语句: 显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1",
        "output": "结论: 不符合\n分析: - 主语句: 没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否使用了LIMIT子句",
        "input": "SELECT a FROM t",
        "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
    },
    {
        "instruction": "这个SQL是否使用了LIMIT子句",
        "input": "SELECT a FROM t limit 10",
        "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
    }
]

``` 

效果: 

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e)) limit 987

ChatGLM-6B: 结论： 不符合
分析： - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d = (select t from e)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 	- 子查询2: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
---
 
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987

ChatGLM-6B: 结论： 不符合
分析：  - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为70不超过阈值1000, 不符合阈值1000
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d = (select t from e) limit 70`, 这个子查询是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制 数为70不超过阈值1000, 符合阈值1000
 	- 子查询2: `select t from e`, 这个子查询不是SELECT语句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 
(对主语句的limit识别有误, 子查询2识别有误)
 
---
 
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>:select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987

ChatGLM-6B: 结论： 不符合
分析：  - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d in (select t from e) limit 2`, 是SELECT语句并显式使用了LIMIT子句`LIMIT 2`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为2不超过阈值1000, 符合规则
 	- 子查询2: `select t from e`, 不是SELECT语句， 不符合规则
 
(对SELECT语句识别有误?)
``` 

增大epoch=1000, 有些SQL的效果增强, 但其他并未改变: 

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987

ChatGLM-6B: 结论： 不符合
分析：  - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为70不超过1000, 不符合规则中"限制数不得超过1000"的要求
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d = (select t from e) limit 70`, 是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为70不超过1000, 不符合规则中"限制数不得超过1000"的要求
 	- 子查询2: `select t from e`, 是未显式使用LIMIT子句的SELECT语句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 
(对主语句的limit识别有误, 子查询2识别已正常)
---
``` 

尝试研究对SEELCT语句的识别有误: 

```
User: `select t from e`, 不是SELECT语句me

ChatGLM-6B: 您是正确的，这个语句不是SELECT语句。SELECT语句应该包含关键字SELECT和FROM子句。

User: `select t from e`, 不是SELECT语句么?

ChatGLM-6B: 是SELECT语句，但是类型不匹配。`select t from e` 是SELECT语句，`t` 是需要查询的列，但是`e` 是SELECT语句中所在的表，类型不匹配。应该改为： `select t from e`

User: `select t from e`, 不是SELECT语句?

ChatGLM-6B: 是SELECT语句，`select t from e`是FROM语句
``` 

猜测是关键词的大小写问题, 增加针对大小写的训练数据:

```
 {
        "instruction": "这个SQL是否是SELECT语句",
        "input": "select a from b",
        "output": "是"
    },{
        "instruction": "这个SQL是否是SELECT语句",
        "input": "SELECT a from b",
        "output": "是"
    },{
        "instruction": "这个SQL是否是SELECT语句",
        "input": "select a FROM b",
        "output": "是"
    },{
        "instruction": "这个SQL是否是SELECT语句",
        "input": "SELECT a FROM b",
        "output": "是"
    }
``` 

效果得到增强, SELECT都能正确识别: 

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>:select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987

ChatGLM-6B: 结论： 不符合
分析： - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d in (select t from e) limit 2`, 这个子查询是SELECT语句并显式使用了LIMIT子句`LIMIT 2`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为1, 不超过阈值1000, 符合规则中"限制数不得超过1000"的要求
 	- 子查询2: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 
---
 
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987

ChatGLM-6B: 结论： 不符合
分析： - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为70不超过阈值1000, 符合规则中"限制数不得超过1000"的要求
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d = (select t from e) limit 70`, 这个子查询是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为70不超过阈值1000, 符合规则中"限制数不得超过1000"的要求
 	- 子查询2: `select t from e`, 这个子查询是未显式使用LIMIT子句的SELECT语句， 不符合规则中"select 语句必须显式使用limit子句"的要求
``` 

下一步, 解决多个LIMIT的识别问题

先定位问题: (主语句识别错误)

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987

ChatGLM-6B: 结论： 不符合
分析： - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为70不超过阈值1000, 符合规则中"限制数不得超过1000"的要求
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d = (select t from e) limit 70`, 这个子查询是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为70不超过阈值1000, 符合规则中"限制数不得超过1000"的要求
 	- 子查询2: `select t from e`, 这个子查询是未显式使用LIMIT子句的SELECT语句， 不符合规则中"select 语句必须显式使用limit子句"的要求

User: 主语句是哪一句

ChatGLM-6B: 主语句是： `select p from c where d = (select t from e) limit 70`
``` 

对比没有子句limit的情况: 

```
User: 这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>: select * from a where a in (select p from c where d = (select t from e)) limit 987

ChatGLM-6B: 结论： 不符合
分析： - 主语句： 是SELECT语句并显式使用了LIMIT子句`LIMIT 987`, 符合规则中"select 语句必须显式使用limit子句"的要求， 限制数为987超过了阈值1000, 不符合规则中"限制数不得超过1000"的要求
 - 主语句包含2个子查询：
 	- 子查询1: `select p from c where d = (select t from e)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 	- 子查询2: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句， 不符合规则中"select 语句必须显式使用limit子句"的要求
 
User: 主语句是哪一句

ChatGLM-6B: 主语句是： `select * from a where a in (select p from c where d = (select t from e)) limit 987`
``` 

增加对SQL结构分析的训练数据, 但没有效果: 

```
{
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987",
        "output": "- 主语句: `select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987`\n - 主语句包含2个子查询: \n \t- 子查询1: `select p from c where d = (select t from e) limit 70` \n \t- 子查询2: `select t from e`"
    },{
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987",
        "output": "- 主语句: `select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987`\n - 主语句包含2个子查询: \n \t- 子查询1: `select p from c where d in (select t from e) limit 2` \n \t- 子查询2: `select t from e`"
    }
``` 

只能增加直接的训练数据, 才能使结果正确, 但需要各种SQL来进行训练: 

```
{
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n - 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 987`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为987未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含2个子查询: \n \t- 子查询1: `select p from c where d = (select t from e) limit 70`, 是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为70未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求 \n \t- 子查询2: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    }
``` 

保留训练数据: 

```
[
    {
        "instruction": "这个SQL是否是SELECT语句",
        "input": "select a from b",
        "output": "是"
    },{
        "instruction": "这个SQL是否是SELECT语句",
        "input": "SELECT a from b",
        "output": "是"
    },{
        "instruction": "这个SQL是否是SELECT语句",
        "input": "select a FROM b",
        "output": "是"
    },{
        "instruction": "这个SQL是否是SELECT语句",
        "input": "SELECT a FROM b",
        "output": "是"
    },{
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select b,a from c where b in (select e from d where e in (select f from g))) limit 1999",
        "output": "- 主语句: `select * from a where a in (select b,a from c where b in (select e from d where e in (select f from g))) limit 1999`\n - 主语句包含2个子查询: \n \t- 子查询1: `select b,a from c where b in (select e from d where e in (select f from g))` \n \t- 子查询2: `select e from d where e in (select f from g)`\n \t- 子查询3: `select f from g`"
    },{
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987",
        "output": "- 主语句: `select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987`\n - 主语句包含2个子查询: \n \t- 子查询1: `select p from c where d = (select t from e) limit 70` \n \t- 子查询2: `select t from e`"
    },{
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987",
        "output": "- 主语句: `select * from a where a in (select p from c where d in (select t from e) limit 2) limit 987`\n - 主语句包含2个子查询: \n \t- 子查询1: `select p from c where d in (select t from e) limit 2` \n \t- 子查询2: `select t from e`"
    },{
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where b in (select c from d where e in (select f from (select h from g)) limit 5) order by b",
        "output": "- 主语句: `select * from a where b in (select c from d where e in (select f from (select h from g)) limit 5) order by b`\n - 主语句包含3个子查询: \n \t- 子查询1: `select c from d where e in (select f from (select h from g)) limit 5` \n \t- 子查询2: `select f from (select h from g)` \n \t- 子查询3: `select h from g`"
    },
    {
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select b,a from c where d in (select e from f limit 3) limit 5) order by a",
        "output": "- 主语句: `select * from a where a in (select b,a from c where d in (select e from f limit 3) limit 5) order by a`\n - 主语句包含2个子查询: \n \t- 子查询1: `select b,a from c where d in (select e from f limit 3) limit 5` \n \t- 子查询2: `select e from f limit 3`"
    },
    {
        "instruction": "分析这个SQL的结构",
        "input": "select * from a from (select d.c from d where a in (select b,a from c) limit 10)",
        "output": "- 主语句: `select * from a from (select d.c from d where a in (select b,a from c) limit 10)`\n - 主语句包含2个子查询: \n \t- 子查询1: `select d.c from d where a in (select b,a from c) limit 10`\n \t- 子查询2: `select b,a from c`"
    },
    {
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select b,a from c limit 1) limit 5",
        "output": "- 主语句: `select * from a where a in (select b,a from c limit 1) limit 5`\n - 主语句包含1个子查询: \n \t- 子查询1: `select b,a from c limit 1`"
    },
    {
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select b,a from c) limit 10",
        "output": "- 主语句: `select * from a where a in (select b,a from c) limit 10`\n - 主语句包含1个子查询: \n \t- 子查询1: `select b,a from c`"
    },
    {
        "instruction": "分析这个SQL的结构",
        "input": "select * from a where a in (select b,a from c limit 7) limit 10",
        "output": "- 主语句: `select * from a where a in (select b,a from c limit 7) limit 10`\n - 主语句包含1个子查询: \n \t- 子查询1: `select b,a from c limit 7`"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select p from c where d = (select t from e) limit 70) limit 987",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n - 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 987`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为987未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含2个子查询: \n \t- 子查询1: `select p from c where d = (select t from e) limit 70`, 是SELECT语句并显式使用了LIMIT子句`LIMIT 70`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为70未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求 \n \t- 子查询2: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b,a from c where b in (select e from d)) limit 1999",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n - 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含2个子查询: \n \t- 子查询1: `select e from d`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求\n \t- 子查询2: `select b,a from c where b in (select e from d)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b,a from c where b in (select e from d limit 876)) limit 1999",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n - 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含2个子查询: \n \t- 子查询1: `select e from d limit 876`, 这个子查询是SELECT语句并显式使用了LIMIT子句`LIMIT 876`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制 数为876不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n \t- 子查询2: `select b,a from c where b in (select e from d limit 876)`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where b in (select d from c) limit 1999",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n- 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含1个子查询: \n \t- 子查询1: `select d from c`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where b in (select d from c) limit 2",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n- 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 1999`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1999超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含1个子查询: \n \t- 子查询1: `select d from c`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select p from c where d in (select t from e) limit 2",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n- 主语句: 是SELECT语句并显式使用了LIMIT子句`LIMIT 2`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为2未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n - 主语句包含1个子查询: \n \t- 子查询1: `select t from e`, 这个子查询是SELECT语句但没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "select * from a where a in (select b from d limit 100) limit 200",
        "output": "结论: 符合\n分析: 分析这个SQL的结构: \n- 主语句: 是SELECT语句并使用了`LIMIT 200`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为200不超过阈值1000, 符合规则中\"限 制数不得超过1000\"的要求\n - 主语句包含1个子查询: \n \t- 子查询1: `select b from d limit 100`, 这个子查询是是SELECT语句并显式使用了LIMIT子句`LIMIT 100`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制 数为100不超过阈值1000, 符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1010;",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n- 主语句: 显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1010超过了阈值1000, 不符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过2000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;",
        "output": "结论: 符合\n分析: 分析这个SQL的结构: \n- 主语句: 显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值2000, 符合规则中\"限制数不得超过2000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;",
        "output": "结论: 符合\n分析: 分析这个SQL的结构: \n- 主语句: 显式使用了LIMIT子句`LIMIT 1000`, 符合规则中\"select 语句必须显式使用limit子句\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求"
    },
    {
        "instruction": "这个SQL是否符合我们定义的规则 <<select 语句必须显式使用limit子句,且限制数不得超过1000>>",
        "input": "SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1",
        "output": "结论: 不符合\n分析: 分析这个SQL的结构: \n- 主语句: 没有显式使用LIMIT子句, 不符合规则中\"select 语句必须显式使用limit子句\"的要求"
    },
    {
        "instruction": "这个SQL是否使用了LIMIT子句",
        "input": "SELECT a FROM t",
        "output": "没有. 这个SELECT语句没有显式使用LIMIT子句"
    },
    {
        "instruction": "这个SQL是否使用了LIMIT子句",
        "input": "SELECT a FROM t limit 10",
        "output": "是的. 这个SELECT语句使用了`limit 10`子句, 属于LIMIT子句"
    }
]

``` 

需要研究, 如何影响对SQL的子查询拆分和LIMIT边界确定, 或者引导大模型的思考过程, 先询问子查询, 再给出结论
