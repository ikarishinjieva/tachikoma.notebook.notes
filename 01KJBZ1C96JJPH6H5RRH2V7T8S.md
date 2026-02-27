---
title: 20230619 - 大模型chatglm微调, 样本对模型的影响 (1)
confluence_page_id: 2392178
created_at: 2023-06-19T10:42:35+00:00
updated_at: 2023-06-20T12:08:11+00:00
---

# 背景

在之前的测试中, 使用SQL审核的QA数据集对chatglm进行微调, 发现有些结果明显偏离训练数据, 发生结果冲突. 举例: 

训练数据的一组: 

```
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须带limit\"的要求, 限制数为1000未超过了阈值, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
``` 

未加载微调时的结果: 

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 这段 SQL 语句并没有违反规则 <<select 语句必须带limit，且限制数不得超过1000>>。

SQL 语句中的LIMIT 子句用于限制返回的结果集数量，而该语句并没有使用该子句。因此，该语句并没有违反规则。

该语句只是SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000; 它将返回 table1 表中所有符合条件的行，数量为 1000。
``` 

加载微调时的结果: 

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 符合规则中"select 语句必须带limit"的要求， 但限制数1000违反了"限制数不得超过1000"的要求
高亮： `SELECT *
FROM table1
WHERE data->'key1' = 'value1'
AND data->'key2' > 1
<b>LIMIT 1000</b>;`
 
 
------
 
(不带分号)
 
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 违反了规则中"select 语句必须带limit"的要求
高亮： `SELECT *
FROM table1
WHERE data->'key1' = 'value1'
AND data->'key2' > 1
<b>LIMIT 1000</b>`
``` 

# 从头研究微调数据的影响

训练数据: 四条一样的数据

```
  [{
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须带limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须带limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须带limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须带limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  }]
``` 

训练命令 (ChatGLM-Efficient-Tuning): 

```
CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://127.0.0.1:7890 python src/train_sft.py     --do_train     --dataset sql_audit     --finetuning_type lora     --output_dir path_to_sft_checkpoint.sql_audit     --per_device_train_batch_size 1     --gradient_accumulation_steps 4     --lr_scheduler_type cosine     --logging_steps 10     --save_steps 1000     --learning_rate 5e-5     --num_train_epochs 100     --fp16
``` 

训练结果: 

```
[INFO|trainer.py:1786] 2023-06-19 10:30:56,406 >> ***** Running training *****
[INFO|trainer.py:1787] 2023-06-19 10:30:56,406 >>   Num examples = 4
[INFO|trainer.py:1788] 2023-06-19 10:30:56,406 >>   Num Epochs = 100
[INFO|trainer.py:1789] 2023-06-19 10:30:56,406 >>   Instantaneous batch size per device = 1
[INFO|trainer.py:1790] 2023-06-19 10:30:56,406 >>   Total train batch size (w. parallel, distributed & accumulation) = 4
[INFO|trainer.py:1791] 2023-06-19 10:30:56,406 >>   Gradient Accumulation steps = 4
[INFO|trainer.py:1792] 2023-06-19 10:30:56,406 >>   Total optimization steps = 100
[INFO|trainer.py:1793] 2023-06-19 10:30:56,407 >>   Number of trainable parameters = 3,670,016
{'loss': 1.9206, 'learning_rate': 4.900734214192358e-05, 'epoch': 10.0}
{'loss': 1.0406, 'learning_rate': 4.567701435686404e-05, 'epoch': 20.0}
{'loss': 0.4445, 'learning_rate': 4.0322676341324415e-05, 'epoch': 30.0}
{'loss': 0.0943, 'learning_rate': 3.346844800613229e-05, 'epoch': 40.0}
{'loss': 0.0085, 'learning_rate': 2.578526897695321e-05, 'epoch': 50.0}
{'loss': 0.0035, 'learning_rate': 1.802522234901927e-05, 'epoch': 60.0}
{'loss': 0.0024, 'learning_rate': 1.0947915553696742e-05, 'epoch': 70.0}
{'loss': 0.0021, 'learning_rate': 5.24612469060774e-06, 'epoch': 80.0}
{'loss': 0.0019, 'learning_rate': 1.4779807761443636e-06, 'epoch': 90.0}
{'loss': 0.0019, 'learning_rate': 1.233599085671e-08, 'epoch': 100.0}
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 100/100 [01:20<00:00,  1.26it/s][INFO|trainer.py:2053] 2023-06-19 10:32:16,760 >>

Training completed. Do not forget to share your model on huggingface.co/models =)

{'train_runtime': 80.3532, 'train_samples_per_second': 4.978, 'train_steps_per_second': 1.245, 'train_loss': 0.3520446900092065, 'epoch': 100.0}
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 100/100 [01:20<00:00,  1.24it/s]
***** train metrics *****
  epoch                    =      100.0
  train_loss               =      0.352
  train_runtime            = 0:01:20.35
  train_samples_per_second =      4.978
  train_steps_per_second   =      1.245
``` 

重点: 

  - epoch = 100
  - train_loss = 0.352

问答结果: 

````
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过900>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 符合规则中"select 语句必须带limit"的要求， 限制数为1000未超过了阈值900, 符合规则中"限制数不得超过900"的要求

(能理解规则中的数值是阈值, 但不能理解限制数和阈值间的大小关系)
------
 
User: SQL <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过900>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`LIMIT 1000`子句， 符合规则中"select 语句必须带limit"的要求， 限制数为1000未超过了阈值900, 符合规则中"限制数不得超过900"的要求
 
(是否使用"SQL"和分号, 不影响结果. 猜测是epoch比较多, 强化记忆)
 
------
User: <<SELECT * FROM table1>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过900>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`LIMIT 0`子句， 没有超过规则中限制数不得超过900的要求， 符合规则要求
 
(解释性错误, SELECT不带LIMIT 等同于 使用了`LIMIT 0`)
 
------
 
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1;>> 是否违反了规则 <<select 语句必须带limit,且限制数不得超过900>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`data->'key1'`和`data->'key2'`关联符进行了高亮显示，并设置了`LIMIT 1`子句，限制了结果集数量为1，符合规则中"select 语句必须带limit"的要求，未超过限制数900，符合规则中"限制数不得超过900"的要求
高亮显示的`SELECT *`语句未使用`LIMIT`子句，不符合规则中"select 语句必须带limit"的要求
```SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1;
LIMIT 1;```

(解释性错误, 原SQL中去掉LIMIT, 大模型会认为SQL带有`LIMIT 1`子句)
```` 

# 问题拆解1 - 对"使用LIMIT"的误解

ChatGLM认为 所有SQL都"使用了limit" (可能是指的SQL文档中的Limit 默认值), 但不是"显式使用了limit"

```
---

User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1;>> 是否使用了limit

ChatGLM-6B: 使用了limit。

该SELECT语句使用了`limit`子句，返回了满足特定条件的未查询记录的行数。`limit`子句指定了要返回的行数的最大值，`0`表示不返回任何记录，`1`表示返回第一行记录，`2`表示返回第二行记录，以此类推。

因此，该SELECT语句返回了`key1`为`'value1'`、`key2`大于等于`1`的记录，共`2`行。

---

User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1;>> 是否显式使用了limit

ChatGLM-6B: 没有显式使用 limit 子句。

``` 

但当修正了规则中的"显式使用"的描述, 分析中可以判断出没有"使用LIMIT", 但仍然符合规则中的要求, 应该是训练数据全是正向数据的问题

```
---

User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1;>> 是否违反了规则 <<select 语句必须显式使用LIMIT,且限制数不得超过900>>

ChatGLM-6B: 结论： 不违反
分析： 该SELECT语句使用了`data->'key1' = 'value1'`和`data->'key2' > 1`的子查询，并没有使用`LIMIT`关键字，限制数也未超过900，符合规则中"select 语句必须显式使用LIMIT"的要求
高亮： `SELECT *
FROM table1
WHERE data->'key1' = 'value1'
AND data->'key2' > 1;`
``` 

修正训练数据集, 将2/4变成负向数据: 

```
  [{
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1 LIMIT 1000;>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 不违反\n分析: 该SELECT语句使用了`LIMIT 1000`子句, 符合规则中\"select 语句必须显式使用limit\"的要求, 限制数为1000未超过了阈值1000, 符合规则中\"限制数不得超过1000\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1\n<b>LIMIT 1000</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 违反\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n//////"
  },
  {
    "instruction": "<<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>",
    "input": "",
    "output": "结论: 违反\n分析: 该SELECT语句没有显式使用`LIMIT`, 不符合规则中\"select 语句必须显式使用limit\"的要求\n高亮: `SELECT *\nFROM table1\nWHERE data->'key1' = 'value1'\nAND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`\n//////"
  }]

``` 

测试效果: 

```
User: <<SELECT * FROM table1 WHERE data->'key1' = 'value1' AND data->'key2' > 1>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有显式使用`LIMIT`, 不符合规则中"select 语句必须显式使用limit"的要求
高亮： `SELECT *
FROM table1
WHERE data->'key1' = 'value1'
AND data->'key2' > 1 <b>[此处应使用LIMIT]</b>;`
//////
 
------
 
User: <<SELECT *\nFROM table1\nWHERE column1 IN (\n  SELECT column2\n  FROM table2\n  WHERE column3 = 3)\n)>> 是否违反了规则 <<select 语句必须显式使用limit,且限制数不得超过1000>>

ChatGLM-6B: 结论： 违反
分析： 该SELECT语句没有显式使用`LIMIT`, 不符合规则中"select 语句必须显式使用limit"的要求
高亮： `SELECT *
FROM table1
WHERE column1 IN (\n  SELECT column2
  FROM table2
  WHERE column3 = 3\n)`
//////
``` 

#
