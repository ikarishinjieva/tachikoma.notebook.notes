---
title: 20230608 - 为sqle选择SQL引擎大模型
confluence_page_id: 2392135
created_at: 2023-06-08T07:52:01+00:00
updated_at: 2023-06-08T14:12:49+00:00
---

# 候选列表

按榜单(<https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard>) 顺序: 

  - <https://huggingface.co/eachadea/vicuna-7b-1.1>
  - <https://huggingface.co/AlekseyKorshuk/vicuna-7b>
  - <https://huggingface.co/tiiuae/falcon-7b>
  - <https://huggingface.co/mosaicml/mpt-7b>  
  

spider排行: <https://yale-lily.github.io/spider>

# refine工具

<https://github.com/OpenAccess-AI-Collective/axolotl>

# DIN-SQL分析

DIN-SQL是spider榜单上的方案, 使用openAI的接口实现

<https://github.com/MohammadrezaPourreza/Few-shot-NL2SQL-with-prompting/blob/main/DIN-SQL.py>

特点: 

在Text-to-SQL的过程中, 在prompt中给予了样例, 使得模型按样例进行模仿. 样例中, 还给出了思考过程, 给予模型以指导

# SQL数据集

  - <https://huggingface.co/SEBIS/code_trans_t5_small_source_code_summarization_sql_multitask_finetune>

# TODO

  - <https://github.com/DorBernsohn/CodeLM/tree/main/SQLM>
    - 直接用t5作fine tuning好像效果不好
    - 使用这个fine tuning
  - <https://github.com/OhadRubin/SmBop>
    - text-to-sql: 效果好像还可以
  - chatglm-6b
    - 效果: 

```
我有一个SQL:
—
select id from exist_db.exist_tb_1 where 2=2;

—

需要你判断SQL是否遵循以下规则:
- 禁止使用没有where条件的sql语句
- 禁止使用where 1=1等无意义的条件

告诉我哪个SQL哪部分不符合哪个规则
 
 
>>>>>>
 
根据您的规则，这个SQL语句不符合以下两个规则：

1. 禁止使用没有where条件的sql语句：

这个SQL语句中使用了where子句，但是并没有使用任何有意义的条件。如果不需要过滤任何数据，可以省略where子句。

2. 禁止使用where 1=1等无意义的条件：

这个SQL语句中使用了where子句，但是条件是2=2，这是一个有意义的条件，因为2是数字，等于2表示它们相等。如果不需要过滤任何数据，可以省略where子句。

因此，这个SQL语句符合所有规则。
```

    - 等待fine tuning
