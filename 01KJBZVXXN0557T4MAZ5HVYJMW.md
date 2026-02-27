---
title: 20250727 - 进行SQL优化的GRPO - 对completion进行分析
confluence_page_id: 4161736
created_at: 2025-07-26T16:45:10+00:00
updated_at: 2025-08-01T06:35:07+00:00
---

# 背景

在 [20250715 - 重新进行SQL优化的GRPO] 中, 尝试12 的训练效果并不好, 分析一下数据completion的结果, 目标是分析出训练的趋势

# 使用脚本统计completion

commit: 8b534eed6b7239dd805018a45b7b0e43236fbda2

脚本: analyze_completions.py

校验结果: 

```
开始比较query...
================================================================================
Query比较分析结果
================================================================================
train_data.grpo.jsonl中的query总数: 187
completions.jsonl中的prompt总数: 135
在train_data中存在但在completions中不存在的query数量: 53
在completions中找到匹配的query数量: 134
``` 

发现训练只使用了134个数据, (还有一个是校验数据), 怀疑是dynamic_sample的问题

# 从低分completion发现的问题

### 问题1: SQL中有DDL, 不符合目标场景: 

```
WITH cte AS (SELECT DISTINCT linked_course_id, course_id, course_name, description FROM (SELECT linked_course_id, course_id, course_name, description FROM online_courses INNER JOIN course ON online_courses.linked_course_id = course.course_id ORDER BY course_id, description, linked_course_id) AS subquery) ALTER TABLE online_courses ADD COLUMN linked_course_id TEXT REFERENCES course(course_id);
``` 

### 问题2: GT中, 对表名的引用层次错误: 

SQL: 

```
WITH cte AS (SELECT Major, count(*) as major_count, StudentID, 1 as extra_column, 'extra_value' as new_column FROM Student GROUP BY Major, StudentID) SELECT Major, major_count FROM cte GROUP BY Major HAVING major_count BETWEEN 2 AND 30
``` 

GT:

```
3. 对(子语句-1.1)应用优化规则: [删除(子语句-1.1)的列\"1 as extra_column\", 删除(子语句-1.1)的列\"'extra_value' as new_column\"]
``` 

模型predict:

```
- 对(子语句1.1)应用优化规则: [删除(CTE定义-1.1.1)的列"extra_column", 删除(CTE定义-1.1.1)的列"new_column"]
``` 

而模型predict对表的引用是正确的

### 问题3: GT中, SQL本身的意义错误, 举例: 

SQL:

```
SELECT * FROM ( SELECT Name AS highest_name, Stadium_ID, Average AS avg_score, Capacity AS extra_column FROM stadium GROUP BY Stadium_ID ORDER BY 100.0 * Average / Capacity DESC LIMIT 1 ) JOIN ( SELECT COUNT ( * ) AS concerts_in_highest_stadium FROM stadium A JOIN concert B ON A.Stadium_ID = B.Stadium_ID WHERE A.Stadium_ID = ( SELECT Stadium_ID FROM stadium GROUP BY Stadium_ID ORDER BY 100.0 * Average / Capacity DESC LIMIT 1 ) ) JOIN ( SELECT Name AS lowest_name FROM stadium GROUP BY Stadium_ID ORDER BY 100.0 * Average / Capacity ASC LIMIT 1 ) JOIN ( SELECT COUNT ( * ) AS concerts_in_lowest_stadium FROM stadium A JOIN concert B ON A.Stadium_ID = B.Stadium_ID WHERE A.Stadium_ID = ( SELECT Stadium_ID FROM stadium GROUP BY Stadium_ID ORDER BY 100.0 * Average / Capacity ASC LIMIT 1 ) )
``` 

这个SQL应该是劣化时不合理

### 问题4: 存在多SQL的case, 举例: 

SQL:

```
WITH cte AS (SELECT referenceRecords.citingpaperid, COUNT(*) AS citation_count, MAX(referenceRecords.citedpaperid) AS max_cited_id, MIN(referenceRecords.citedpaperid) AS min_cited_id FROM referenceRecords GROUP BY referenceRecords.citingpaperid ORDER BY citation_count DESC NULLS LAST) SELECT referenceRecords.citingpaperid, citation_count FROM cte; SELECT p.paperid, p.numciting FROM paper p WHERE p.numciting > 0 ORDER BY p.numciting DESC; SELECT p.title, COUNT(referenceRecords.citedpaperid) AS num_cited_papers FROM paper p JOIN referenceRecords ON p.paperid = referenceRecords.citingpaperid GROUP BY p.title ORDER BY num_cited_papers DESC;
``` 

### 问题5: GT中出现了非法的格式, 举例: 

SQL:

```
select ( select lab.labresult, lab.labid from lab where lab.patientunitstayid in ( select patient.patientunitstayid from patient where patient.patienthealthsystemstayid in ( select DISTINCT patient.patienthealthsystemstayid from patient where patient.uniquepid = '027-203413' and patient.hospitaldischargetime is null order by patient.patienthealthsystemstayid ) order by patient.patientunitstayid ) and lab.labname = 'pao2' order by lab.labresulttime desc limit 1 ) - ( select lab.labresult from lab where lab.patientunitstayid in ( select patient.patientunitstayid from patient where patient.patienthealthsystemstayid in ( select patient.patienthealthsystemstayid from patient where patient.uniquepid = '027-203413' and patient.hospitaldischargetime is null ) ) and lab.labname = 'pao2' order by lab.labresulttime asc limit 1 )
``` 

GT: 表名加了引号, 这格式比较小众

```
- 对(\"子查询-1.1\")应用优化规则: [删除(\"子查询-1.1\")的列\"lab.labid\"]
``` 

### 问题6: 模型不容易理解"EXCEPT子查询-1.4.1.1"这种描述. 举例: 

SQL:

```
SELECT B.product_id , B.product_name , SUM ( A.order_quantity ) AS quantity , SUM ( A.order_quantity * B.product_price ) AS revenue , SUM ( A.order_quantity * B.product_price ) * ( 1.0 - 40.0 / 100 ) AS profit FROM Sales_Records A JOIN Products B ON A.product_id = B.product_id WHERE B.product_name = 'Apple' GROUP BY B.product_id UNION SELECT B.product_id , B.product_name , SUM ( A.order_quantity ) AS quantity , SUM ( A.order_quantity * B.product_price ) AS revenue , SUM ( A.order_quantity * B.product_price ) * ( 1.0 - 20.0 / 100 ) AS profit FROM Sales_Records A JOIN Products B ON A.product_id = B.product_id WHERE B.product_name = 'gucci' GROUP BY B.product_id UNION SELECT B.product_id , B.product_name , SUM ( A.order_quantity ) AS quantity , SUM ( A.order_quantity * B.product_price ) AS revenue , SUM ( A.order_quantity * B.product_price ) * ( 1.0 - 50.0 / 100 ) AS profit FROM Sales_Records A JOIN Products B ON A.product_id = B.product_id WHERE B.product_name != 'Apple' AND B.product_name != 'gucci' GROUP BY B.product_id UNION SELECT B.product_id , B.product_name , 0 , 0 , 0 FROM ( SELECT product_id, product_name, product_description, category_id FROM Products A EXCEPT SELECT B.product_id FROM Sales_Records A JOIN Products B ON A.product_id = B.product_id ) A JOIN Products B ON A.product_id = B.product_id
``` 

其中EXCEPT子句的名称是: "EXCEPT子查询-1.4.1.1"

GT没问题, 模型推理: 

```
4. 对(子查询1.4.1.1)应用优化规则: [删除(子语句-1.4.1)的列"product_name", 删除(子语句-1.4.1)的列"product_description", 删除(子语句-1.4.1)的列"category_id"]  
``` 

应当增强这方面微调数据

### 问题7: INSERT语句

先分析到这里

# 粗暴的处理

将目前无法得到最高分1.0的case进行淘汰.

使用脚本: /data/huangyan/sql-ai-2/make_train_dataset/filter_high_reward_cases.py

筛选completion文件: /data/huangyan/sql-ai-2/train_logs/2025-07-25_11-27-08/train_output/v0-20250725-112857/completions.jsonl 

  - [20250715 - 重新进行SQL优化的GRPO] 中 尝试12 的checkpoint

筛选出 最高分=1.0的数据, sft共80条, grpo共100条.

LR=1e-4, 训练效果: 

![image2025-7-28 10:16:27.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-28%2010%3A16%3A27.png)

可以看到train/reward上升. eval/reward也有上升趋势, 但不明显

再次分析completions文件, 发现: 

  - 有一些case是错误的, 跟之前的分析的问题类似
  - 有一部分是比较复杂的case, case本身没错

再次将completions文件筛选, 并训练: 

![image2025-7-28 23:6:16.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-28%2023%3A6%3A16.png)

train/reward 能到0.5, eval/reward也有少量提升

再次检查completions文件: 只有一个数据的最大Reward达不到1.0, 其他数据都能达到. 

尝试抑制KL爆炸, 将beta=0.04 提升到 beta=0.06, 增加epoch=24, 训练效果 (commit: 6c13e391239ac5d1cb8b6becadea1ddc6413562e): 

![image2025-7-29 11:30:0.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-29%2011%3A30%3A0.png)

train/reward略微提升, 但eval/reward彻底趴窝, 没有增长趋势. 

将beta降低到0.02, 并且设置dynamic_sample=False, 查看completion是否覆盖到所有数据, 并且查看eval/reward的趋势

# 设置dynamic_sample=False, 跑一个epoch, 看一下completion是否能覆盖所有训练数据

跑了两个epoch, 看到completion文件中的数据有32个prompt (训练数据中有75个), 并且分布不均匀

还是得仔细研究一下GRPO的数据选择算法

分析结论: [20250729 - 对GRPO的completion文件和step/epoch计算的理解]

# 设置 overlong_filter=false, dataset_shuffle=false, beta=0.02, dynamic_sample=False 进行训练, 并对数据进行增长分析

设置 overlong_filter=false, dataset_shuffle=false. 主要是为了研究: [20250729 - 对GRPO的completion文件和step/epoch计算的理解]

研究 每个prompt分组, 训练前后的平均分, 是否有增长, 来判断训练的方向

commit: e6d58c808b6f6ea4ce84c294e3ce9ad566245abb

训练效果: 

![image2025-7-29 18:37:5.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-29%2018%3A37%3A5.png)

train/reward可以达到0.8. eval/reward很糟糕.

分析评分劣化的case, 比如: 

![image2025-7-29 17:54:18.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-29%2017%3A54%3A18.png)

![image2025-7-29 17:54:39.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-29%2017%3A54%3A39.png)

查看劣化原因: 

  - 会进行多次大块的重复输出: [1.txt](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/1.txt)
  - 出现不合规的表名, 比如: 删除(分母子查询-1.1的子查询)的列"*"
  - 出现语句级的重复输出, 且生成被截断: [2.txt](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/2.txt)  

对验证数据进行劣化检查, 得到类似的问题, 都是因为重复输出导致劣化. 

对于过长的输出, 注意观察clipped_ratio

对于重复输出的问题, 是否要检查训练数据? 

# 增强数据

对于"从低分completion发现的问题"中的问题:

  - 通过对SQL进行正则检查 , 修复 1/4/7
  - 通过LLM对SQL的合法性进行检查, 修复 3
  - 将动作解析出 (语句名, 列名), 在结构中查找语句名, 获取"SQL示意", 然后在其中查找列名. 修复问题2/5
    - 问题2 不一定能完全修复, 如果父结构和子结构都包含完整的SQL示意, 那么有可能定位到父结构 而不是 子结构

# 使用增强后的数据进行训练

训练效果不佳: 

![image2025-7-29 23:33:35.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-29%2023%3A33%3A35.png)

分析completion文件, 发现问题: 

  1. 有一些GT错误, 不止删除了列, 还简化了表
  2. 大量的completion评分低, 是因为产生了无限重复的输出

考虑: 

  1. 增加sft的epoch
     1. 检查sft后的eval: 'eval_loss': 0.4427253, 'eval_token_acc': 0.88125555
  2. 检查训练数据, 出现了非法数据: 

```
    ...

    11. 检查所有的优化是否改变了SQL的结果, 以及优化后的SQL是否是一个能正确运行的SQL
    检查的日志:
    - 原始SQL的子查询中, `DISTINCT`关键字作用于`(flno, origin, destination, airline, 1)`这几个列的组合。
    - 由于`WHERE`子句已经限定了`origin`和`destination`, 这等价于对`(flno, airline)`进行去重。
    - 优化后的SQL的子查询中, `DISTINCT`关键字仅作用于`flno`列。
    - 考虑一种情况: 如果同一个航班号`flno`（例如'UA123'）在同一航线（旧金山到纽约）下，被不同的航空公司（`airline`）记录了多次，原始查询会因为`airline`列的不同而将这些记录视为不同的行，并在最终结果中返回多次该`flno`。
    - 而优化后的查询仅对`flno`去重，无论有多少家航空公司运营，该`flno`都只会出现一次。
    - 因此，优化移除了`DISTINCT`子句中的列，改变了去重逻辑，从而可能导致最终输出的行数减少，改变了SQL的查询结果。
    检查结论: \"优化改变了SQL结果\"
    
    1. 对整个SQL从外往内分析语句: 先分析(主语句-1), 然后分析(子查询-1.1)

2. 对(主语句-1)进行列引用分析: (主语句-1)的`SELECT`语句引用了(子查询-1.1)的列\"flno\", `ORDER BY`子句也引用了列\"flno\"。

3. 对(主语句-1)应用优化规则: []

4. 对(子查询-1.1)进行列引用分析: (子查询-1.1)的列\"flno\"被(主语句-1)引用。列\"origin\", \"destination\", \"airline\", \"1 AS extra_column\"未被(主语句-1)引用。
```

```
    7. 最终优化完成。

8. 优化后的完整SQL:
-----------
SELECT count(*)>0 FROM (SELECT DISTINCT patient_id, 1 as extra_column FROM patient WHERE patient.uniquepid = '030-49739' ORDER BY patient_id, extra_column) AS subquery
-----------

6. 检查是否可以再次应用优化规则
    {
      \"检查的日志\": \"对优化后的SQL中间版本进行分析: SELECT count(*)>0 FROM (SELECT DISTINCT patient_id, 1 as extra_column FROM patient WHERE patient.uniquepid = '030-49739' ORDER BY patient_id, extra_column) AS subquery。\
      1. 外层查询为`count(*)`，不依赖子查询的任何具体列。\
2. 子查询中的列为`patient_id`和`extra_column`。\
3. `patient_id`列虽然未在外层查询中使用，但在子查询自身的`ORDER BY patient_id, extra_column`子句中被引用，因此不能删除。\
4. `extra_column`列虽然未在外层查询中使用，但在子查询自身的`ORDER BY patient_id, extra_column`子句中被引用，因此不能删除。\
5. 子查询中所有列均因为在`ORDER BY`子句中被使用而无法被优化规则删除。\",
      \"检查结论\": \"不能再次应用优化规则\"
    }
    
    7. 检查所有的优化是否改变了SQL的结果, 以及优化后的SQL是否是一个能正确运行的SQL
    {
      \"检查的日志\": \"1. 原始SQL的最终目的是通过`count(*)>0`判断是否存在符合条件的记录，只关心子查询返回的行数是否大于0，不关心具体行数和内容。\
      2. 在子查询中，被删除的列是`uniquepid`和`'extra' as added_column`。\
3. 由于`WHERE`子句已经限定`patient.uniquepid = '030-49739'`，因此在子查询的结果集中，`uniquepid`列的值对于所有行都是一个常量。\
4. `'extra' as added_column`列本身就是一个常量。\
5. `DISTINCT`关键字作用于所有选定列的组合。在原始SQL中，`DISTINCT`作用于`(patient_id, constant_uniquepid, constant_extra_column, constant_added_column)`。其去重效果等价于`DISTINCT`作用于`(patient_id, constant_extra_column)`，因为其他列都是常量，对去重没有贡献。\
6. 优化后的SQL子查询是`SELECT DISTINCT patient_id, 1 as extra_column ...`，其返回的行数与原始SQL的子查询完全相同。\
7. 由于子查询返回的行数没有改变，外层查询`count(*)>0`的结果也不会改变。\
8. 优化后的SQL语法正确，`ORDER BY`子句引用的列`patient_id`和`extra_column`依然存在于`SELECT`列表中，SQL可以正常执行。\",
      \"检查结论\": \"优化没有改变SQL结果且优化后的SQL是一个正确的SQL\"
    }
    
    9. 检查所有的优化是否存在超出优化规则的规定的情况
    {
      \"检查的日志\": \"1. 优化规则为“删除子查询中无意义的列, 这些列在外查询中没有被使用”。\
      2. 本次优化删除了子查询中的`uniquepid`和`'extra' as added_column`两列。\
3. 检查`uniquepid`列：该列未在外层查询`SELECT count(*)>0 ...`中被使用。因此，删除该列符合规则。\
4. 检查`'extra' as added_column`列：该列也未在外层查询中被使用。因此，删除该列符合规则。\
5. 对于未被删除的`patient_id`和`extra_column`列，虽然它们也未在外层查询中使用，但由于它们在子查询的`ORDER BY`子句中被引用，删除它们会导致SQL语法错误。因此，保留这些列是为了保证优化后SQL的正确性，这种处理方式是对规则的合理应用，并未超出规则的规定。优化操作仅限于规则所描述的范围：删除外层未使用的、且删除后不影响SQL正确性的列。\",
      \"检查结论\": \"优化没有超出规则规定\"
```

```
    
    10. 检查所有的优化是否存在超出优化规则的规定的情况
    检查日志:
    1.  本次优化应用的规则是\"删除无用的列查询\", 其规定是删除子查询中未被外层查询引用的列。
    2.  在本次优化中, 除了删除了`dp.publication_date`, `dp.title`, `dk.keyword`这三个无用列(这符合规则)之外, 还一并移除了子查询(CTE)中的`ORDER BY d.did, dp.publication_date`子句。
    3.  移除`ORDER BY`子句是一个关联优化, 但该操作本身并不属于\"删除列\"的范畴, 而是对SQL子句的修改。
    4.  因此, 移除`ORDER BY`子句的操作超出了\"删除无用的列查询\"这一规则的字面规定。
    检查结论: \"优化存在超出规则规定的情况\"
    
    好的, 我将续写你的思考步骤, 并严格按照优化规则和输出格式要求完成任务。

### 思考与优化过程

1.  **对整个SQL从外往内分析语句:** 先分析(主语句-1), 然后分析(子查询-1.1)。

2.  **对(主语句-1)进行列引用分析:** (主语句-1)是一个`SELECT`语句, 其数据源是名为`cte`的公共表表达式, 该表达式的结果来自于(子查询-1.1)。(主语句-1)在其`SELECT`列表和`GROUP BY`子句中引用了`cte`的以下列: `did`, `pid`, `kid`。
```

  3. 对输出长度进行惩罚? 

增加了sft的epoch, 手工删除了非法数据, 训练效果: 

![image2025-7-30 10:4:43.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-30%2010%3A4%3A43.png)

从clipped_ratio看上去, 长度仍然会炸, 认为是这组数据导致的这个现象 (这组数据是第一次用gemini-2.5-pro进行端到端生成, 以前用的主题是deepseek-v3)

尝试1: 增加长度惩罚

增加惩罚参数: 

```
    --reward_funcs sql_ai_action_match soft_overlong \    
	--reward_weights 1.0 0.5 \
    --soft_max_length 6400 \
    --soft_cache_length 4096 \
``` 

压制了train/completions/clipped_ratio, 但eval部分没有压制

![image2025-7-30 12:49:49.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-30%2012%3A49%3A49.png)

尝试2: 让大模型对训练数据进行重新装修

增加脚本: analyze_sql.v4.py

commit: a748761b9e2f4e042a1addad393e083ac3012b2a

效果: 

![image2025-7-30 14:12:41.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-30%2014%3A12%3A41.png)

reward都很低, 主要问题是对动作的描述不符合规范, 比如出现了: "对 `子查询-1.2.1` 应用优化规则" (其中分隔符不对, 应当是括号).

检查训练数据, 并没有发现类似的问题. 

但因为增加了对训练数据的重新装修和验证, 训练数据的数据量变少 (28条), 导致格式固定没有那么稳定? 

  - 尝试调低LR = 5e-5, 增加sft的epoch=28
    - ![image2025-7-30 15:39:44.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-30%2015%3A39%3A44.png)  

  - 需要增加数据量, 增加训练数据到42条, 并(调低LR = 5e-5, 增加sft的epoch=28):
    - ![image2025-7-31 0:49:2.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-31%200%3A49%3A2.png)
  - 增加数据量, 增加训练数据到71条, 并(调低LR = 5e-5, 增加sft的epoch=28):
    - ![image2025-8-1 9:48:59.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-8-1%209%3A48%3A59.png)
    - 随着数据量增加, train/reward有良好的上升趋势. 但eval/reward提升不大
      - 总共运行
    - 先将train/reward调优. 研究completion的问题: 
      - reward可以达到1.0, 但随着训练下降:
        - 有GT错误, 将"1 as col"的引用写成"col"
      - 无法达到reward=1.0:
        - 对于DISTINCT的判断, 模型无法做到正确的思考. 需要增加DISTINCT数据? 
        - 对DISTINCT的理解不足: 

```
SELECT hospitaladmittime FROM (SELECT DISTINCT hospitaladmittime, uniquepid, patientname, address, phone FROM medical_records WHERE uniquepid = '004-7984' ORDER BY phone, address) subquery
```

          - 其中涉及到根据业务含义 (uniquepid和patientname的业务关系), 推断出 删除"health_records.patientname"选择, 并不影响DISTINCT的结果

```
WITH cte AS (SELECT DISTINCT health_records.hospitaladmitsource, health_records.hospitaladmittime, health_records.patientname, health_records.doctorname FROM health_records WHERE health_records.uniquepid = '015-56556' ORDER BY health_records.hospitaladmittime desc, health_records.patientname, health_records.doctorname) SELECT hospitaladmitsource FROM cte LIMIT 1
``` 
        - 同一个SQL有多个训练数据? 需要检查数据生成脚本
          - 已删除重复的训练数据
      - 还发现训练趋势, 往必要的精简路线上走, 比如: 
        - 训练初期的正确答案

````
                                    <think>

</think>

### 思考与分析过程

1.  **对整个SQL从外往内分析语句:**
    为了应用"删除无用的列查询"规则，我们需要从最外层查询开始，逐层向内分析其对子查询的列依赖关系。分析路径为: `主语句-1` -> `主FROM子语句-1.1`。

2.  **对(主语句-1)进行列引用分析:**
    *   `主语句-1` 的数据源是名为 `subquery` 的子查询结果集 (即 `主FROM子语句-1.1` 的结果)。
    *   分析 `主语句-1` 的 `SELECT` 子句 `AVG(score)`，可知其仅引用了 `subquery` 的 `score` 列。
    *   结论: 外层查询 `主语句-1` 仅依赖其数据源 `subquery` 的 `score` 这一列。

3.  **对(主FROM子语句-1.1)进行列引用分析:**
    *   `主FROM子语句-1.1` 的 `SELECT` 列表为: `score`, `judge_name`, `show_name`, `review_date`。
    *   根据上一步的分析，外层查询仅需要 `score` 列。
    *   结论: `judge_name`, `show_name`, 和 `review_date` 是无用列，可以被删除。

4.  **对(主FROM子语句-1.1)应用优化规则: [删除(主FROM子语句-1.1)的列"judge_name", 删除(主FROM子语句-1.1)的列"show_name", 删除(主FROM子语句-1.1)的列"review_date"]**

5.  **优化后的完整SQL:**
    ```sql
    SELECT AVG(score) FROM (SELECT score FROM performance_reviews WHERE judge_name = 'Simon Cowell') AS subquery;
    ```

6.  **检查是否可以再次应用优化规则:**
    *   **检查日志:**
        分析优化后的SQL，`主FROM子语句-1.1` 现在仅 `SELECT` `score` 列。`主语句-1` 使用了 `score` 列。不存在任何在子查询中被选择但未在外层查询中使用的列。
    *   **检查结论:** "不能再次应用优化规则"

7.  **检查所有的优化是否改变了SQL的结果, 以及优化后的SQL是否是一个能正确运行的SQL:**
    *   **检查日志:**
        1.  **结果一致性**: 被删除的列 `judge_name`, `show_name`, `review_date` 未在 `主语句-1` 的聚合函数 `AVG` 计算中使用。删除它们不影响 `WHERE` 子句的过滤逻辑 (因为过滤条件 `judge_name = 'Simon Cowell'` 作用于原始表 `performance_reviews` 而非子查询)。因此，用于计算平均值的 `score` 列集合保持不变，最终结果与原始SQL完全一致。
        2.  **语法正确性**: 优化后的SQL语法结构完整，子查询和主查询的引用关系明确，是一个可以正确执行的SQL。
    *   **检查结论:** "优化没有改变SQL结果且优化后的SQL是一个正确的SQL"

8.  **检查所有的优化是否存在超出优化规则的规定的情况:**
    *   **检查日志:**
        本次优化操作为"删除子查询中无意义的列, 这些列在外查询中没有被使用"。我们精确地删除了 `主FROM子语句-1.1` 中未被 `主语句-1` 引用的 `judge_name`, `show_name`, `review_date` 列。该操作完全符合规则定义，未涉及任何其他修改。
    *   **检查结论:** "优化没有超出规则规定"
                                
````

  

        - 训练末期的正确答案

````
                                    <think>

</think>

### 对(主FROM子语句-1.1)进行列引用分析:
- (主FROM子语句-1.1)的列"score"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"judge_name"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"show_name"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"review_date"被(主FROM子语句-1.1)引用

### 对(主FROM子语句-1.1)应用优化规则: [删除(主FROM子语句-1.1)的列"judge_name", 删除(主FROM子语句-1.1)的列"show_name", 删除(主FROM子语句-1.1)的列"review_date"]

### 对(主FROM子语句-1.1)进行列引用分析:
- (主FROM子语句-1.1)的列"score"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"judge_name"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"show_name"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"review_date"被(主FROM子语句-1.1)引用

### 对(主FROM子语句-1.1)应用优化规则: [删除(主FROM子语句-1.1)的列"judge_name", 删除(主FROM子语句-1.1)的列"show_name", 删除(主FROM子语句-1.1)的列"review_date"]

### 对(主FROM子语句-1.1)进行列引用分析:
- (主FROM子语句-1.1)的列"score"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"judge_name"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"show_name"被(主FROM子语句-1.1)引用
- (主FROM子语句-1.1)的列"review_date"被(主FROM子语句-1.1)引用

### 对(主FROM子语句-1.1)应用优化规则: [删除(主FROM子语句-1.1)的列"judge_name", 删除(主FROM子语句-1.1)的列"show_name", 删除(主FROM子语句-1.1)的列"review_date"]

### 优化后的完整SQL
```sql
SELECT AVG(score) FROM (SELECT score FROM performance_reviews WHERE judge_name = 'Simon Cowell') AS subquery;
```
                                
````

  

      - 另外, 在一些(效果不好的)答案中, 出现了一些乱码/混乱:

```
                                    <think>

</think>

1. 对(语句1)从外往内分析语句话：  
语句结构为 `VARCHAR cte FOR (子查询-1.1) SELECT flno FROM cte`。  
  
2. 对(child语句1.1)从小向回收分析语句：  
子引用语句结构为 `VARCHAR cte FOR (子查询-1.1`SupportedContent) SELECT flno FROM cte`。  
  
对(子查询-1.沃尔玛)重新进行引用分析：携程携程结构为 SELECT fld, origin, destination, airline, 1 AS extra_column FROM air_volume_records WHERE origin = '洛杉矶'	AND destination = '纽约携程"。  

3. 对(CultureInfo1.1.1)继续分析：  
引用的列有 `flare`, ` originate`, `destination`, `"airline"`, `1 AS extra_column"。

- 在上一级语句中 (子语句1.1)
- 国内的列 flno 存储，但是 origin、destination、airlene、extra_column 这些列在上一条语境中没有被应用。
- 判别这些在上级语语境中不需要查询列
  - origin
  - destination
  - airline
  - 1 AS additional column

4. 对(dllexport1.1)应用优化-rule  
对(CTE-1.1)应用 optimize规则：[^]  
对(CultureInfo1.1.1)应用优化规定：[删除(子query1.1.1)的列"origin", 删除(subquery1.卡通.1)的列"delivery", 删除(subquery1.1.1)的字段 "airline", 删除(subquery1.东方财富.1)的列"good extra COLUMN]  

5. 优化完整的.sql  
Optimization after：  
WITH cte AS (SELECT flav, NULL AS origin, NULL AS destination, NULL AS airport, 1 ASCOLUMN FROM air_volume_records WHERE owner = '洛杉矶' DISCLAIM destination = ' New York ' )SELECT fla FROM cote WHERE 1 = 1

7. 对(子语句子查询1.东方财富.1)应用重新优化-rule：  
- 对(子查询1.1乒乓)应用优化规则：[删除(子语一句1.1乒乓}的列"originate", 删除(子语 sentenced1携程1.东方财富.1)的列"profile destination", 删除(子语 sentence1.携程1.东方财富.1)的隱私權 "airline", 删除(子语 sentence1.携程1.福田.1)的列"1 AS EXTRA COLUMN"]  

8. 港区对(语句1)进行复盘及分析：  
在主语句中，只需求出航班号(flno)列，origin, destination, aviation, expert_column 初在后续分析中未引用，这些列均可在(subquery1.1.1)中学删除，不影响最终结果携程，同时增强查询疗效及优化数据库效能.

9. 优化后查询 SQL 更加精致，减少了传递及查询列，使得 query 渲染更清晰。
                                
```

  

    - 为什么评估和训练的方向不同, 研究评估的completion
      - SQL的结构中, 对于同一个子结构的名字描述不同, 导致错乱: 
        - ![image2025-7-31 14:55:40.png](/assets/01KJBZVXXN0557T4MAZ5HVYJMW/image2025-7-31%2014%3A55%3A40.png)  
  

      - 出现乱码, 跟训练数据类似:   

```
### 对(子查询v1-1.1.1.1)应用 optimization 觶: [hapus(子查询v1-1.1.1.1)의 columna "sum(bingoggl) als bingu", 删除(子查询v1-1.1.1.1)的列" Lauderdale" ]
```

  

      - 重新生成训练数据. 然后进行评估: 

```
CUDA_VISIBLE_DEVICES=0 swift infer     --adapters /data/huangyan/sql-ai-2/train_logs/2025-07-31_01-13-49/train_output/v0-20250731-011531/checkpoint-1632   --infer_backend pt     --temperature 0.6     --max_new_tokens 6000  --val_dataset /data/huangyan/sql-ai-2/train_logs/2025-07-31_01-13-49/val_data.sft.jsonl     --max_batch_size 1 
```

  

  

        - 对last_checkpoint进行评估: [附件: checkpoint_1632_temp_0.txt] [附件: checkpoint_1632_temp_0_6.txt] 

          - 没有正确的结果, 思考很简单, 认为是过拟合导致

        - 对eval比较好的checkpoint=690进行评估: 
          - [附件: checkpoint_690_temp_0.txt] [附件: checkpoint_690_temp_0_6.txt]   

          - 没有完全正确的结果, 有接近正确的结果, 思考简单

# TODO

  - warmup阶段会导致reward大幅下降, 考虑将warmup阶段拉长, 或去除
  - 在训练数据中: 
    - 计算列分析/表连接分析/列引用分析 的出现次数为: 65/64/210
    - 但在评估中, 绝大部分都是 列引用分析, 其他两种步骤出现的很少
  - 目前解决了训练数据拟合的问题, 需要考虑如何解决训练数据效果好, 但评估数据很差的gap
