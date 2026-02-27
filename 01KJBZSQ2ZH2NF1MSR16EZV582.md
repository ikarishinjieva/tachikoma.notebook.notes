---
title: 20250513 - 研究SQL优化模型微调 - 扩展Rule-5
confluence_page_id: 3801924
created_at: 2025-05-13T06:26:12+00:00
updated_at: 2025-05-13T11:50:44+00:00
---

# 训练1

commit: 380add4ac01ad96f8f6f1a20bb1ea9e322eea18e

训练数据: 

  1. 可以安全展开子查询: 50条
  2. 因为外部查询最高运算优先级 > 子查询最低运算优先级, 所以不能安全地展开子查询: 20条
  3. 因为外部查询将子查询结果与其他表连接, 所以不能安全地展开子查询: 20条  
  

```
    {
        "id": 8,
        "violate_desc": """
    1. 在SQL某处新增一个子查询, 并给予一个别名, 这个子查询的结果与其他表进行连接, 这个子查询可能是能被展开到父查询中的""",
        "violate_key": "新增的子查询别名",
        "violate_analyze": """
1. 结论应当是: 因为外部查询将子查询结果与其他表连接, 所以不能安全地展开子查询 \"{violate_value}\"
""",
        "rewrite_rule": rule_5,
        "allowed_sql_regex": r"^\s*(SELECT|WITH)\b",
        "rule_checker": None,
        "max_data": 20
    },
    {
        "id": 7,
        "violate_desc": """
    1. 在SQL某处新增一个子查询, 并给予一个别名, 这个子查询中包括GROUP BY/ORDER BY/等高级运算. 这个子查询可能是能被展开到父查询中的""",
        "violate_key": "新增的子查询别名",
        "violate_analyze": """
1. 结论应当是: 因为外部查询最高运算优先级 > 子查询最低运算优先级, 所以不能安全地展开子查询 \"{violate_value}\"
""",
        "rewrite_rule": rule_5,
        "allowed_sql_regex": r"^\s*(SELECT|WITH)\b",
        "rule_checker": None,
        "max_data": 20
    },
    {
        "id": 6,
        "violate_desc": """
    1. 在SQL某处新增一个子查询, 并给予一个别名, 这个子查询应当是能被展开到父查询中的""",
        "violate_key": "新增的子查询别名",
        "violate_analyze": """
1. 结论应当是: 可以安全地展开子查询 \"{violate_value}\"
""",
        "rewrite_rule": rule_5,
        "allowed_sql_regex": r"^\s*(SELECT|WITH)\b",
        "rule_checker": None,
        "max_data": 50
    },

``` 

评估数据集: [data_0005 (1).xlsx](/assets/01KJBZSQ2ZH2NF1MSR16EZV582/data_0005%20%281%29.xlsx)

评估结果: 

  - case 1: 改写正确
  - case 2: 
    - 第一轮: 逻辑错误: "因为子语句满足可展开条件，不需改写。"
    - 第二轮: 改写正确
  - case 3: 
    - 改写正确
  - case 4: 
    - 改写正确
  - case 5: 
    - 规则错误, 后续分析
  - case 6: 
    - 不需改写, 判断正确
  - case 7: 
    - 不需改写, 判断正确
  - case 8: 
    - 不需改写, 判断正确
  - case 9: 
    - 改写正确
  - case 10: 
    - 改写正确

关于case 5的规则错误分析: 

对于这个SQL: 

```
SELECT 
    count(1) 
FROM 
    (SELECT c_custkey, avg(age) FROM customer group by C_NATIONKEY)
AS 
    derived_t1;
``` 

根据重写规则, 子查询最低运算优先级为3（包含GROUP BY和聚合函数）, 外部查询最高运算优先级为3（对子查询的列使用了聚合函数）, 所以判断子查询可以展开. 

但根据SQL含义是不能展开的. 其中的核心矛盾是: 

  - 如果子查询使用了GROUP BY, 已经用分组改变了原始表的行, 如果父查询要在分组结构上再次改变行数 (比如使用GROUP BY, count(1)等), 那子查询就无法展开

找到另一个反例: <https://poe.com/s/SJDf5ljj2jSPHDj85eTE>

  - 可扩展成自动化的流程

需要修订规则:

  - 一个想法: 将明确的简单场景总结出来, 变成若干个定向的规则. 对复杂的场景不做优化

# 整理现状

rule 1/3/5 都可以正确判断,

总结做数据的流程: (make_sql_analyze_data.json_format.ipynb)

  1. 对SQL进行劣化 (add_violate_in_sql)
  2. 对劣化SQL生成 SQL结构故事 (make_sql_story)
  3. 在 SQL结构故事 中, 增加对表连接的分析 (add_join_analyze_in_sql_story)
  4. 在 SQL结构故事 中, 增加对计算列的分析 (add_calc_col_analyze_in_sql_story)
  5. 在 SQL结构故事 中, 增加改写分析 (add_rewrite_analyze_in_sql_story)
  6. 在 SQL结构故事 中, 检查并修正 各章节是否与 其父语句给出的信息 冲突 (fix_sql_story_by_parent_section)
  7. 再次进行步骤6的检查并修正 (修正后再次检查/修正) (fix_sql_story_by_parent_section)
  8. 检查并修正 分析的自洽, 包括: (fix_sql_story_by_logic_finesss)
     1. 其中的"分析"是否是自洽的

     2. 其中的"分析"从中文语法的角度, 是否是完备且明确的

     3. 其中的"分析"是否存在逻辑完整性缺失

  9. 检查并修正 改写质量, 包括: (fix_sql_story_by_correction)
     1. 改写后是否会导致表连接的错误
     2. 改写后是否会导致表计算列的错误
     3. 改写后是否会导致SQL执行错误
  10. 检查并修正 措辞 (fix_sql_story_by_wording), 包括: 
     1. \- 提到"查询"时, 必须明确其指的是"语句"还是"结果字段"  
\- 在"需要做的改写"中, 指令必须要**明确**, 说明:   
\- 明确的指令(明确了操作类型, 对象名称, 规格等): 添加列abc, 添加索引def  
\- 不明确的指令(没有指定明确的对象): 添加符合条件的列, 并添加响应索引  
\- 明确的指令(应指明修改动作): 删除了表xyz  
\- 不明确的指令(描述的不是修改动作): 仅保留表xyz  
\- 当描述"X被语句使用"时, 必须明确指出"X被语句的什么部分使用"

  11. 检查并修正 "改写前"和"改写后"的出现时间
  12. 检查并筛选: 改写是否改变了整个SQL的最终结果
  13. 检查并筛选: 改写是否严格符合规则描述, 不多不少
  14. 检查并筛选: 改写是否符合 "数据增强"的方向要求

TODO:

  1. 稳定性不好, 需要训练使其稳定
  2. 需要多个回合进行迭代优化, 需要训练至一次完成
  3. 多规则整合到一个模型
  4. 梳理结构的信息过多, 可以精简 (GRPO)
  5. 长SQL
