---
title: 20250821 - SQL优化的GRPO训练 - 重新制作数据生成过程
confluence_page_id: 4358270
created_at: 2025-08-21T14:54:48+00:00
updated_at: 2025-09-21T23:19:31+00:00
---

在之前 ([20250813 - 继续进行SQL优化的GRPO训练]) 的训练中, 有大量的时间在每个训练数据分析中 reward的异常原因, 大部分原因是因为 GT错误, 或者 因为匹配问题导致reward的计算失误

新的思路: 

  - 确定输入的SQL和输出的正确SQL
  - 根据输入和输出, 生成中间的思考过程 (相当于将思考过程 框在夹具中, 防止GT错误)
  - 针对易发生的匹配问题, 要求模型进行过程的修正重写

使用新的数据生成脚本 /data/huangyan/sql-ai-2/make_train_dataset.v3/make_dataset.py :

  - 只生成思考过程
  - 根据输入和输出生成

## 训练1

  - 使用20个数据(随机挑选), 进行GRPO训练: (作为基线)
    - ![image2025-8-22 10:22:7.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-22%2010%3A22%3A7.png)
    - 分析completion:
      - 达不到reward=1.0: prompt 1/2/3/4/5
        - prompt 1: 不同表的同名列 区分不开
        - prompt 2: 即使给定输入和输出SQL, GT的思考过程也是错误的
          - SQL: 

```
select j.xingming, y.xingming as yisheng_xingming from (select id, xingming, shoujihao from (select id, xingming, shoujihao, substring(wenti, 1, 50) as wenti_jianyao from (select id, xingming, shoujihao, wenti from jiankangzixun where riqi > '2023-01-01') as t1) as t2) as j join (select id, xingming from yisheng where zhuanye = 'neike') as y on j.id in (select jiankangzixun_id from huifu where yisheng_id = y.id);
``` 
        - prompt 3: 不同表的同名列 区分不开
        - prompt 4: 不同表的同名列 区分不开
        - prompt 5: 即使给定输入和输出SQL, GT的思考过程也是错误的
          - SQL: 

```
SELECT consultation_id, consultation_date FROM (SELECT consultation_id, consultation_date, consultation_duration FROM (SELECT consultation_id, consultation_date, consultation_duration, consultation_fee FROM TeleConsultation WHERE consultation_date > '2023-01-01') AS subquery1 UNION ALL SELECT consultation_id, consultation_date, consultation_duration FROM (SELECT consultation_id, consultation_date, consultation_duration, consultation_fee FROM TeleConsultation WHERE consultation_date <= '2023-01-01') AS subquery2) AS final_query ORDER BY consultation_date;
``` 
      - 长期曲线较低: prompt 9/14/17  

        - 暂不分析, 先解决以上两个问题
  - 使用20个数据(长度最长的), 进行GRPO训练: (绿色曲线, 蓝色为基线)
    - ![image2025-8-22 10:24:2.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-22%2010%3A24%3A2.png)
    - 训练集上的表现和基线差不多, 评估集在step=60后下降
  - sft使用80条数据, GRPO使用(长度最长的)20个数据: (红色曲线, 蓝色为基线)
    - ![image2025-8-22 10:25:51.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-22%2010%3A25%3A51.png)
    - 训练集初始值很高, 没有改进空间
    - 评估集在step=40后下降
  - sft和GRPO都使用80条数据
    - ![image2025-8-22 15:17:56.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-22%2015%3A17%3A56.png)
    - 训练数据学习曲线极值比 基线 更优
    - 评估集学习曲线很低

# 训练2 - 解决"即使给定输入和输出SQL, GT的思考过程也是错误的"

使用脚本: /data/huangyan/sql-ai-2/make_train_dataset.v3/check_dataset.py 检查思考过程是否匹配输出SQL, 发现12/86条错误数据, 需要修复

在生成脚本中, 增加检查逻辑

对现有数据集, 将错误数据删除, 保留正确数据, 然后补充数据, 凑够80条

commit: 803be19d5e2ce29983c0690917280751b68af3ae

使用20个数据(随机挑选), 进行GRPO训练: 红色曲线

![image2025-8-22 17:39:58.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-22%2017%3A39%3A58.png)

  - 训练reward趋势良好, 比基线高
  - 评估崩溃

训练数据中出现容易引起歧义的描述, 而训练过程受到影响: 

![image2025-8-22 16:41:35.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-22%2016%3A41%3A35.png)

对训练数据进行如下修订: 

  1. 不对 主语句的各查询字段 进行检查, 消除歧义
  2. 将"同级或父级语句" 改成 "同级其他语句 或 父级各级语句"
  3. "没有被同级其他语句 或 父级各级语句 (这里改成各个语句列表)引用, 可以删除"
  4. 对SQL引用的片段进行要求: 保持原格式, 可使用"..."来省略SQL结构, 突出重点

# 训练3 - 修订数据

根据训练2的结论, 重新生成训练数据

commit: 0265c561cec6361a72c10ea5b8ff6db43407fc53

### 使用20个数据(随机挑选), 进行GRPO训练: (蓝色为训练1的极限, 白色为本次训练):

![image2025-8-22 23:43:49.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-22%2023%3A43%3A49.png)

  - 训练曲线, 与极限类似, 但最后一段出现下降
  - 评估曲线, 优于基线
  - 分析completion
    - 最高奖励无法达到极值: prompt 1/2/3/4
      - prompt 1: GT分析过程错误:

```
select sf.pinpai, avg(sf.jiage) as avg_jiage from (select pinpai, jiage from (select pinpai, yanse, jiage from (select pinpai, yanse, jiage from shafa where yanse = 'red') as sf_inner1) as sf_inner2) as sf group by sf.pinpai;
```

      - prompt 2: 不同表的同名列 区分不开
      - prompt 3: 不同表的同名列 区分不开
      - prompt 4: 评估数据
        - 检查completion, 大部分在0.8, 偶尔有一些0.0是出现了低级错误
    - reward长期达不到极值: prompt 9/11
      - prompt 9: 已经出现了1.0向好的趋势
      - prompt 11: 大部分是1.0, 有些劣化的case, 是因为分析过程中SQL样例过长, 导致生成被截断

### 使用20个数据(长度最长的), 进行GRPO训练: (蓝色曲线, 白色为随机选择20个数据的训练)

![image2025-8-23 13:42:55.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-23%2013%3A42%3A55.png)

  - 训练阶段和评估阶段, reward都比上一轮要好
    - 分析completion
      - 奖励最大不能到1.0: prompt 1/2/3/4
        - prompt 1: 不同表的同名列 区分不开
        - prompt 2: 不同表的同名列 区分不开
        - prompt 3: 仅一个数据点, 没有参考
        - prompt 4: 评估数据
          - 无法识别级联删除 (GT: "但由于上层子查询v2-1.1.3中cost未被使用，可以删除")
      - 奖励长期达不到1.0: prompt 10/15/17
        - prompt 10: 会多一个"删除", 是对UNION父语句进行的重复删除
        - prompt 15: 训练最后已经出现了频繁的1.0
        - prompt 17: 训练最后已经出现了频繁的1.0
      - 还有多组数据, 是在训练结束时才达到1.0
  - checkpoint: /data/huangyan/sql-ai-2/train_logs/2025-08-22_22-05-16/train_output/v1-20250822-220803/checkpoint-160
    - 进行续训
      - 使用相同数据
        - 第一次OOM, 重试
        - 训练曲线: (浅蓝色曲线为续训曲线, 深蓝色为初训曲线, 白色为基线)
          - ![image2025-8-24 11:4:13.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-24%2011%3A4%3A13.png)
          - 训练曲线可以继续上升
          - 评估曲线拉平
          - completion:
            - 最大奖励不够1.0: prompt 1/2/3/4/5
              - prompt 1: 不同表的同名列 区分不开
              - prompt 2: 不同表的同名列 区分不开
              - prompt 3: 会多一个"删除", 是对UNION父语句进行的重复删除
              - prompt 4: 多删除了一个"计算列"的成分列
                - "COUNT(p.package_id) AS package_count", 模型会删除子查询的package_id;
                - 形似的引用, "SUM(p.weight_kg) AS total_weight". 模型会正确判断出列的引用"weight_kg"
                - 不知原因
              - prompt 5: 校验数据:
                - 无法识别级联删除 (GT: "但由于上层子查询v2-1.1.3中cost未被使用，可以删除")
        - 使用zero3, 进行续训, 对比一下zero3的影响:
          - ![image2025-8-24 15:37:59.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-24%2015%3A37%3A59.png)
          - 白色曲线是使用zero3, 蓝色曲线是没有使用zero3
          - 对于续训这个场景, 两根线reward趋势类似, 但zero3更平稳 (std更低), 而且总体表现略好
          - 时间对比: 
            - zero3: "epoch": 16.25, "global_step/max_steps": "130/160", "percentage": "81.25%", "elapsed_time": "4h 4m 21s", "remaining_time": "56m 23s"
            - 不用zero3: "epoch": 16.25, "global_step/max_steps": "130/160", "percentage": "81.25%", "elapsed_time": "4h 24m 20s", "remaining_time": "1h 1m 0s"
          - 使用zero3时间略好...?  
  

      - 续训使用新旧数据混合
        - 训练曲线: 
          - ![image2025-8-24 21:27:42.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-24%2021%3A27%3A42.png)
          - 蓝色曲线是之前的续训, 橙色曲线是使用旧数据10个+新数据10个进行续训(zero3)
          - 两根曲线差别不大, 新的20条数据对

### 使用20个数据(长度最长的), 进行GRPO训练, LR调大一倍为4e-5: 

(白色为随机选择20个数据的训练, 蓝色曲线为续训曲线, 绿色曲线为本次训练)

![image2025-8-24 15:58:51.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-24%2015%3A58%3A51.png)

  - 训练阶段: 调大学习率的曲线能增长更好, 接近于续训的效果
  - 评估阶段: 在step=80以后, 评估出现崩溃
  - 需要查看评估数据的completion:
    - step=80之后, 崩溃是因为对 FROM语句的各列不进行检查 (有时候原因是因为 "略过集合语句的检查", 有时候没写原因, 如下图)
    - ![image2025-8-24 16:19:51.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-24%2016%3A19%3A51.png)

由此看, LR过大时, 模型学到了不合适的规则, 所以导致评估崩溃

### sft使用80条数据, GRPO使用(长度最长的)20个数据

  - 报错: An error occurred: CUDA out of memory. Tried to allocate more than 1EB memory
  - 重跑: (橙色曲线, 白色为随机选择20个数据的训练)
    - ![image2025-8-23 19:2:31.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-23%2019%3A2%3A31.png)
    - 训练起点很高 (sft数据充分), 但校验数据起点变低

### sft和GRPO都使用80条数据

绿色曲线为本次训练 (未完), 白色为随机选择20个数据的训练

![image2025-8-23 14:9:44.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-23%2014%3A9%3A44.png)

  - 在训练集上起点很高
  - 在校验集上不及基线

# 训练4 - 解决"不同表的同名列 区分不开" 和 "级联删除"

在之前的几次训练中都出现了"不同表的同名列 区分不开"问题: 

  - 使用20个数据(随机挑选), 进行GRPO训练
  - 使用20个数据(长度最长的), 进行GRPO训练
    - 续训
  - 使用脚本检查了训练数据, 并没有类似的问题, 想不出其他办法

校验数据出现了 级联删除 的问题

  - 无法识别级联删除 (GT: "但由于上层子查询v2-1.1.3中cost未被使用，可以删除")
  - 需要重新构建训练数据, 新增对"级联删除"的转折语句

修订数据生成方法

  - 修复级联删除的问题
  - 在提示词中增加提示"注意区分不同表的同名列"
  - 在生成数据的校验过程中, 需要明确检查"可以删除"字样, 并仅根据这个字样来修改SQL. 目前模型会根据分析过程过度推理来修改SQL. 

重新生成所有数据, 进行训练 (5条数据包含"可以认为" (包含级联删除的特征) + 15条最长的数据, zero3, 一次续训, LR=2e-5)

训练效果: 

![image2025-8-25 10:19:43.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2010%3A19%3A43.png)

  - 续训对于reward的作用不大
  - 校验reward达到新高
  - 分析completion:
    - 第一阶段训练结束:
      - 最大reward不到1.0: prompt 1/2/3/4
        - prompt 1: 在UNION结构中, 模型错误地判断子语句的选择列是被UNION父语句引用了, 其实是被 子语句本身作为了连接列

```
SELECT DISTINCT p.line_name, q.defect_rate, q.measurement_timestamp FROM t_production_lines p JOIN (SELECT m.machine_id, m.production_line_id, q.defect_rate, q.measurement_timestamp, q.batch_number FROM t_machine_data m JOIN t_quality_metrics q ON m.machine_id = q.machine_id WHERE m.status_code = 'OPERATIONAL' UNION ALL SELECT m.machine_id, m.production_line_id, q.defect_rate, q.measurement_timestamp, q.batch_number FROM t_machine_data m JOIN t_quality_metrics q ON m.machine_id = q.machine_id JOIN t_maintenance_logs l ON m.machine_id = l.machine_id WHERE l.maintenance_type = 'PREVENTIVE' AND l.completion_time > DATEADD(month, -1, GETDATE())) q ON p.line_id = q.production_line_id WHERE p.capacity_utilization > 0.7 ORDER BY q.defect_rate DESC, q.measurement_timestamp;
```

        - prompt 2: 无法区分 不同表的同名列 
        - prompt 3: 校验数据: 
          - 模型推理会多一个"删除指令", 是UNION的父语句上多出的"删除指令", 不影响最终结果
        - prompt 4: 
          - 模型推理, 错误地认为COUNT被having引用
          - ![image2025-8-25 11:43:34.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2011%3A43%3A34.png)
          - 在结构分析中, 将COUNT作为了单独的子句
            - ![image2025-8-25 11:43:59.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2011%3A43%3A59.png)  
  

      - reward长期不到1.0: prompt 5
        - 模型推理会多一个"删除指令", 是UNION的父语句上多出的"删除指令", 不影响最终结果
    - 续训结束
      - 最大reward不到1.0: prompt 1/2/3/4/5
        - prompt 1: 出现错误的分析: 

```
动作3: 检查语句(UNION第二部分-1.1.2)的查询字段(m.machine_id): 被 同级其他语句或父级各级语句(子查询q-1.1)作为(表连接列)引用: `ON p.line_id = q.production_line_id`
```

        - prompt 2: 模型推理会多一个"删除指令", 是UNION的父语句上多出的"删除指令", 不影响最终结果
        - prompt 3: 无法区分 不同表的同名列 
        - prompt 4: 校验数据: 模型推理会多一个"删除指令", 是UNION的父语句上多出的"删除指令", 不影响最终结果
        - prompt 5: 模型推理, 错误地认为COUNT被having引用
      - 本轮的错误, 与上一轮基本一致, 没有任何进步. 模型在PPO过程中找不到新的方向

检查训练数据: 

  - GT错误: 主语句规则使用错误
    - ![image2025-8-25 14:58:10.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2014%3A58%3A10.png)
  - GT错误: 错误地展开了UNION主语句的字段: 
    - ![image2025-8-25 15:0:11.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2015%3A0%3A11.png)
  - 分析GT:
    - 5/9/10: 主语句为UNION
    - 14/15/16: 子语句为UNION
      - 14: 子语句为UNION, GT错误 (错误地展开了UNION主语句的字段)

# 训练5 - 在训练4基础上, 使用30条数据

![image2025-8-25 14:19:25.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2014%3A19%3A25.png)

绿色线为 使用30条数据的第一段训练, 红色线为使用20条数据的第一段训练 (训练4)

  - 两条训练曲线的趋势差不多
  - 使用30条数据时, 评估集崩溃
  - 检查completions: 
    - 评估集: 后半段reward都为0, 原因是 对于"主FROM子语句-1.1"的检查, 没有检查任何字段, 而是直接检查子查询, 导致问题: 
      - ![image2025-8-25 14:24:34.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2014%3A24%3A34.png)
    - 检查一下各训练数据, 是否有类似数据 (正则: 各查询字段[^\\\\]*(\\\n)+[^\\\\]*各查询字段)
      - 有一组数据出现类似问题: 

```
<think>

</think>

- 动作1: 从外往内, 逐个语句进行检查: (主语句-1, 主FROM子语句-1.1, 子查询过滤国家-1.1.1, 数据关联主表-1.1.1.1, 区域信息表-1.1.1.2, 分类信息表-1.1.1.3)
  - 动作2: 检查语句(主语句-1)的各查询字段: 根据优化规则, 不需要检查主语句, 直接检查各个子语句
    - 动作2: 检查语句(主FROM子语句-1.1)的各查询字段: (region, metric_value)
      - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(region): 被 同级其他语句或父级各级语句(主语句-1)作为(选择字段)引用: SELECT region, SUM(metric_value) AS total_value ...
      - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(metric_value): 被 同级其他语句或父级各级语句(主语句-1)作为(计算列)引用: SUM(metric_value) AS total_value
    - 动作2: 检查语句(子查询过滤国家-1.1.1)的各查询字段: (v.region, v.metric_value, r.country, c.description)
      - 动作3: 检查语句(子查询过滤国家-1.1.1)的查询字段(v.region): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(选择字段)引用: SELECT region, metric_value FROM ...
      - 动作3: 检查语句(子查询过滤国家-1.1.1)的查询字段(v.metric_value): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(选择字段)引用: SELECT region, metric_value FROM ...
      - 动作3: 检查语句(子查询过滤国家-1.1.1)的查询字段(r.country): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(条件列)引用: WHERE country = 'USA'
      - 动作3: 检查语句(子查询过滤国家-1.1.1)的查询字段(c.description): 没有 被同级其他语句或父级各级语句(主FROM子语句-1.1, 主语句-1)引用, 可以删除
    - 动作2: 检查语句(数据关联主表-1.1.1.1)的各查询字段: (visualization_data)
      - 动作2: 检查语句(区域信息表-1.1.1.2)的各查询字段: (region_info)
        - 动作2: 检查语句(分类信息表-1.1.1.3)的各查询字段: (category_info)
```

      - 结构分析中, 将JOIN的简单表作为了单独结构, 所以出现了上面的问题 (将表名作为了字段名): 
        - ![image2025-8-25 14:35:37.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2014%3A35%3A37.png)
      - 需要修正这组数据的GT, 再测试?

# 训练6: 修复训练4中数据"错误地展开了UNION主语句的字段"

使用检查脚本, 检查当前训练数据(20条), 有两条GT类似的错误: 

```
"SELECT consultation_id, consultation_date FROM (SELECT consultation_id, consultation_date ..."
"select a.yonghu_ming, a.dengji, c.neirong, c.fabu_shijian from yonghu a join ..."
``` 

修正数据生成脚本, 增加了相关检查.

手工修复数据, 重新训练: 

  - 第一段训练: (蓝色曲线)
    - ![image2025-8-25 23:59:35.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2023%3A59%3A35.png)
    - 评估效果 不如 训练4, 主要原因是 "对 FROM语句的各列不进行检查", 这个错误在调大LR和增加训练数据 两个训练中都出现过
    - 检查训练数据: 
      - 人工检查训练数据, 发现一个错误: 
        - ![image2025-8-25 23:25:26.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-25%2023%3A25%3A26.png)
    - 另外, 在修复数据时, 修改了训练数据的prompt, 但没有修改校验数据的prompt

修改数据: 

  1. 修改校验数据的prompt
  2. 修改错误的数据

再进行训练: 

![image2025-8-26 10:37:19.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-26%2010%3A37%3A19.png)

评估崩溃: 检查评估completion, 大部分问题出现在:

  - \- 动作2: 检查语句(主FROM子语句-1.1)的各查询字段: (v1.months, v1.bingoggr, v1.valid_account, t1.fd_ggr, v3.fd, t1.fd_ggr/v3.fd, v2.cost), 对于集合操作类SQL, 跳过集合操作的父语句, 直接检查各个子语句
  - 错误: 将FROM子语句判断成了集合操作
  - 检查训练数据, 一共有四组数据涉及到集合操作: 11/13/15/16
    - 数据13出现分析错误: 
      - ![image2025-8-26 10:47:54.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-26%2010%3A47%3A54.png)
    - 其他三个数据没有问题

修复以上一条问题数据, 再次训练

(红色曲线为上次训练, 蓝色为本次训练(未完成))

![image2025-8-26 12:54:34.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-26%2012%3A54%3A34.png)

  - 评估集仍然是崩溃的
  - 分析completion: 评估崩溃的原因跟之前类似
  - 将本次的训练数据, 和训练4的训练数据 进行对比, 有两个case的solution不同: 11/14, prompt进行了更新如图: ![image2025-8-26 13:21:54.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-26%2013%3A21%3A54.png)
    - 测试是否是prompt的问题: 使用当前训练的数据, 但将提示词还原
      - 评估曲线仍然是崩溃的
  - 检查之前的训练的日志, 训练4的日志中没有出现 "All completions are overlong and truncated", 但之后eval崩溃的日志中, 都出现了"All completions are overlong and truncated"这行日志, 意味着model长度不够长? 
    - 将model length从10000 调整到12000, 仍然出现overlog
    - 将model length调整到14000, 仍然出现overlog
    - 将model length调整到16000, 仍然出现overlog
    - 检查completion, 发现两种超长情况: 不断重复, 在SQL示例中写了完整SQL.
      - 对于"在SQL示例中写了完整SQL"的情况, 检查训练数据, 没有出现类似情况
    - 将sft的epoch从12调整成16, 在grpo中增加soft_overlong惩罚 ( 4000-10000 柔性惩罚) 
      - 仍然出现overlong
    - 将sft的训练数据增加到30, 保持grpo的训练数据为20
      - 仍然出现overlong, 且评估崩溃
    - 完全使用训练4的数据和参数
      - 仍然出现overlong, 但评估曲线正常
    - 尝试降低temperature
      - temperature=0.3, 浅红色曲线, 仍然出现overlong但数量少
      - temperature=0.6, 绿色曲线, 出现overlong且数量多
      - ![image2025-8-27 9:55:43.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-27%209%3A55%3A43.png)
      - eval都不高
    - 但偶然发现 将sft的训练数据增加到30, 保持grpo的训练数据为20, temperature=0.3, eval的初始效果很好, 训练曲线和训练4差不多, 但曲线一路下行: 
      - ![image2025-8-27 12:48:24.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-27%2012%3A48%3A24.png)
      - 研究eval completion
        - 随着训练进行, 出现了更多的0.0, 都是同一个问题: 最外层的查询的字段分析消失了, 跟之前的问题一样
        - 也就是说, 当前这20条数据, 让GRPO更倾向于学习到 "最外层的查询的字段分析消失" 这个手段
    - 将sft和grpo的训练数据都增加到30, temperature=0.3: 问题依旧
      - ![image2025-8-27 15:41:28.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-27%2015%3A41%3A28.png)

增加reward (Action2FieldCountReward), 检查action2后是否有足够的action3, 针对eval中字段检查缺失的问题

# 训练7: 增加reward (Action2FieldCountReward), 检查action2后是否有足够的action3, 针对eval中字段检查缺失的问题

### 使用30条数据进行sft和grpo, temperature=0.3

![image2025-8-27 23:25:10.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-27%2023%3A25%3A10.png)

  - 评估曲线起点较高 (sft的数据从20条增加到30条后, 起点就比较高), 并且不再崩溃 (增加了Action2FieldCountReward后就不再崩溃)
  - 查看train阶段的Action2FieldCountReward统计值, 统计值并不高
    - 从completion中检查Action2FieldCountReward的详细值: 
      - 总共4290个评分, 其中168个被扣分
        - -0.5共10个, -0.4共3个, -0.3共15个, 等等
      - 也就是说: 一些特别case的评分很高, 但被统计平均掉了, 但这些特别的case对训练的影响很大
  - 分析completion: (/data/huangyan/sql-ai-2/train_logs/2025-08-27_16-38-29/train_output/v1-20250827-164153/completions.jsonl)
    - 评估数据: 有两个问题: UNION的父语句, 并没有被忽略检查, 所以字段使用时在父语句上分析的, 子语句上就认为字段没有被使用
      - ![image2025-8-28 10:38:32.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-28%2010%3A38%3A32.png)
    - 最高reward没有达到1.0的数据: prompt 1/2/3/4/5/6/7/8/9
      - prompt 1: GT错误, 出现了表检查自己的问题: 

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(shebei_zhuangtai): 被 同级其他语句或父级各级语句(主FROM子语句-1.1.1)作为(条件列)引用: ... where shebei_zhuangtai = 1
```

      - prompt 2: GT错误, 出现了表检查自己的问题
      - prompt 3: GT格式错误
      - prompt 4: 推理中, 错误地展开了UNION父语句的分析
      - prompt 5: GT错误, 少分析了一个字段
      - prompt 6: GT错误, 错误地判断了级联删除
        - (子查询中的soil_type, 被外层排序语句引用, 不能删除. 但GT判定为级联删除)

```
SELECT plot_id, planting_date FROM (SELECT plot_id, planting_date, soil_type FROM (SELECT plot_id, planting_date, soil_type, yield FROM t_farm_plot WHERE farm_id = 123) AS sub1 ORDER BY soil_type DESC) AS sub2 WHERE YEAR(planting_date) = 2023;
``` 
      - prompt 7: 推理中, 错误地展开了UNION父语句的分析
      - prompt 8: 推理会多删除一个字段, 该字段被父级的COUNT函数引用
      - prompt 9: 推理分析错误, SQL片段和分析不一致: 

```
- 动作3: 检查语句(子查询nx-1.2)的查询字段(gongchang_id): 被 同级其他语句或父级各级语句(主语句-1, 子查询ns-1.1, 子查询ng-1.3)作为(表连接列)引用: ns.xiangmu_id = nx.id
```

    - reward长期没有到1.0: prompt 21
      - (是一个长SQL) 对于UNION的子语句, 没有分析其包含的下一级子语句

### 使用30条数据进行sft, 使用20条数据进行grpo, temperature=0.3

(蓝色曲线是本次训练, 红色曲线是上次训练, 作为基线)

![image2025-8-28 10:17:23.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-28%2010%3A17%3A23.png)

  - 训练曲线趋势 与 基线类似
  - eval的曲线趋势方向下降
  - 检查completion:
    - 评估数据: 后期出现多个0.0, 原因跟之前一样, 忽略了对外层查询字段的分析
    - 最高reward没有达到1.0的数据: prompt 1/2/3/4/5
      - prompt 1: GT错误, 出现了表检查自己的问题
      - prompt 2: GT错误, 错误地判断了级联删除
      - prompt 3: 推理中, 错误地展开了UNION父语句的分析
      - prompt 4: 推理会多删除一个字段, 该字段被父级的COUNT函数引用
      - prompt 5: 错误处理了 不同表的同名列

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3

![image2025-8-28 10:18:59.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-28%2010%3A18%3A59.png)

  - 训练曲线趋势 与 基线类似
  - eval的曲线一直不高

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.9

![image2025-8-28 18:41:1.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-28%2018%3A41%3A1.png)

  - completion分析
    - 有一个训练数据的GT有错误, 检查层级错误: 
      - 两端分析的 父级/子级 关系 是冲突的

```
- 动作3: 检查语句(内层子查询-1.1.1)的查询字段(actual_yield_kg): 被 同级其他语句或父级各级语句(UNION第一部分-1.1)作为(计算列)引用: `(pc.actual_yield_kg * 2.20462) AS yield_lbs`
 
 - 动作3: 检查语句(UNION第一部分-1.1)的查询字段(pc.variety_id): 被 同级其他语句或父级各级语句(内层子查询-1.1.1, t_crop_variety)作为(表连接列)引用: `JOIN t_crop_variety cv ON pc.variety_id = cv.variety_id`
``` 
      - 最高reward没有达到1.0的数据: 
        - prompt 1: 推理中, 出现了表检查自己/子表的问题
        - prompt 2: 推理中, 错误地展开了UNION父语句的分析
        - prompt 3: 错误处理了 不同表的同名列
        - prompt 4: 
          - 训练数据对子查询的结构拆分有歧义, 所以分析结果也存在歧义: 需要规范结构定义
            - ![image2025-8-28 19:8:18.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-28%2019%3A8%3A18.png)
      - 长期不达标: prompt 13:  

        - prompt 13: 推理中, 错误地展开了UNION父语句的分析

# 训练8: 增加两个reward

新增两个reward: 

  - Action3ReferenceValidationReward: 检查动作3中的引用语句列表是否合法。确保引用语句不能是检查语句本身或子级语句。
  - Action2StructureComplianceReward: 一个奖励函数，检查action2是否遵循集合操作的处理规则。

    - 如果GT中是 集合操作, 那么推理中也需要是集合操作. 并且其后不应该有action3自操作
    - 如果GT中不是 集合操作, 那么推理中也需要不是集合操作

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.9

红色为基线, 绿色为本次训练

![image2025-8-29 11:33:3.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-29%2011%3A33%3A3.png)

  - 校验曲线很低
  - 检查completion
    - eval: 将最外层引用都判断成了"对于集合操作类SQL, 跳过集合操作的父语句, 直接检查各个子语句"

检查训练数据 (30条), 对奖励有问题的数据进行修正: 

![image2025-8-29 13:7:30.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-29%2013%3A7%3A30.png)

修正后, 有一些数据会有 statement coverage的问题, 核心问题是 在解析结构的过程中, 将一些基础表作为了单独的结构, 但这些基础表不需要分析, 所以导致coverage不足.

先进行训练, 如有相关问题, 再重新生成相关数据

### 使用30条数据进行sft, 使用30条数据进行grpo, temperature=0.9

绿色曲线

![image2025-8-29 15:6:59.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-8-29%2015%3A6%3A59.png)

  - 校验曲线下行
    - 刚开始的评估都正常, 后面更多的0.0数据, 是将外层语句过度划分成了"对于集合操作类SQL, 跳过集合操作的父语句, 直接检查各个子语句"
    - 检查训练数据, 其中有3个数据涉及到"对于集合操作类SQL, 跳过集合操作的父语句, 直接检查各个子语句", 没发现训练数据的问题
    - 考虑将"对于集合操作类SQL, 跳过集合操作的父语句, 直接检查各个子语句"换一种描述形式
    - 或者增加更多的UNION数据

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30

深红色为基线, 灰色为本次训练

![image2025-9-1 13:59:10.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-1%2013%3A59%3A10.png)

  - train/reward 和 eval/reward 都不及基线
  - 分析completion:
    - 最大reward不能达到1.0: prompt 1-7
      - prompt 1: 对于union, 某一个子句都被错误地判断成 被其他子句 使用
        - union在最外层, 所以判断为: "因为该字段是UNION操作的一部分，为最终输出结果集所必需，不能删除"
      - prompt 2: 对于union, 某一个子句都被错误地判断成 被其他子句 使用
      - prompt 3: 对于union的列分析, 分析错误: 

```
- 动作3: 检查语句(UNION第一部分-1.1.1)的查询字段(m.machine_id): 被 同级其他语句或父级各级语句(子查询q-1.1)作为(表连接列)引用: `ON p.line_id = q.production_line_id`
```

      - prompt 4: 对排序列引用判断错误: 

```
- 动作3: 检查语句(内层子查询-1.1.1)的查询字段(soil_type): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(排序字段)引用: `ORDER BY soil_type DESC`
```

      - prompt 5: 对条件列引用判断错误: 

```
- 动作3: 检查语句(诊断子查询-1.1.1)的查询字段(pd.department_id): 被 同级其他语句或父级各级语句(主语句-1)作为(条件列)引用: `AND d.department_id = 'DEP001'`
```

      - prompt 6: 
        - 不同表的同名列判断错误: 

```
- 动作3: 检查语句(子查询nx-1.2)的查询字段(gongchang_id): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: `ns.gongchang_id = ng.id`
```

      - prompt 7: 评估数据: 
        - 有一组UNION的父语句被错误展开
    - 长期不达1.0: prompt 15
      - 删除了UNION子句中凑个数的 null

发现reward函数中, 对action3的解析, 只匹配了有引用的情况, 而没有匹配无引用的情况, 进行修复后, 检查训练数据, 有很多不匹配, 准备进行手工修复

### 使用30条数据进行sft, 使用30条数据进行grpo, temperature=0.3, epoch=30

![image2025-9-1 14:7:53.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-1%2014%3A7%3A53.png)

  - eval/train崩溃

# 训练9: 修复reward函数中, 对action3的解析

commit: 69212603f215af6623b8dadb2665e882cb1d22be

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30

深红色为基线, 浅红色为本次训练

![image2025-9-2 10:7:40.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-2%2010%3A7%3A40.png)

  - train/reward增长形式正常
  - eval/reward中, Action3F1到0.8, 但Action2StructCompliance一直在扣分 (学不到这个知识)
  - 分析completion
    - 最大reward达不到1.0: prompt 1-8
      - prompt 7: 评估数据: 主要问题是将UNION的父语句展开分析了, 导致后续的分析偏差   

        - 需要一个更好的方式处理UNION的父语句?
      - prompt 1: 错误的分析: 
        - 需要检查"被同级其他语句或父级各级语句"引用中的列表

```
- 动作3: 检查语句(主SELECT第一部分-1.1)的查询字段(c.neirong): 没有 被同级其他语句或父级各级语句(主SELECT第二部分-1.2)引用, 可以删除
``` 
      - prompt 2: 错误的分析: 
        - 原因不详, m.machine_id确实是一个JOIN的键, 但这里JOIN的片段是错的, 而且也应当能删除

```
- 动作3: 检查语句(UNION第一部分-1.1.1)的查询字段(m.machine_id): 被 同级其他语句或父级各级语句(主语句-1, 子查询q-1.1, UNION第二部分-1.1.2)作为(表连接列)引用: `JOIN (子查询q-1.1) q ON p.line_id = q.production_line_id`
``` 
      - prompt 3: 
        - 删除了占位用的null

```
    - 动作3: 检查语句(UNION第三部分-1.3)的查询字段(null): 没有 被同级其他语句或父级各级语句(UNION ALL第一部分-1.1, UNION ALL第二部分-1.2)引用, 可以删除
    - 动作3: 检查语句(UNION第三部分-1.3)的查询字段(null): 没有 被同级其他语句或父级各级语句(UNION ALL第一部分-1.1, UNION ALL第二部分-1.2)引用, 可以删除
    - 动作3: 检查语句(UNION第三部分-1.3)的查询字段(null): 没有 被同级其他语句或父级各级语句(UNION ALL第一部分-1.1, UNION ALL第二部分-1.2)引用, 可以删除
``` 
      - prompt 4: 不能区分不同表的同名列
      - prompt 5: GT错误:
        - 这个排序应当不能删除

```
- 动作3: 检查语句(内层子查询-1.1.1)的查询字段(soil_type): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(排序列)引用: ORDER BY soil_type DESC。但由于主FROM子语句-1.1中的soil_type字段可以删除, 可以认为soil_type没有被有效引用, 可以删除
``` 
      - prompt 6: 不能区分不同表的同名列:

```
- 动作3: 检查语句(子查询nx-1.2)的查询字段(gongchang_id): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: `ns.gongchang_id = ng.id`
```

  
  

      - prompt 8: 分析错误, 原因不详
        - guanzhu_count在结构分析中被拆出一个子查询, 但实际上只是一个函数: "COUNT(DISTINCT g.beiguanzhu_id)". 尝试修正GT, 让结构拆解更合理

```
- 动作3: 检查语句(主JOIN子查询-1.2)的查询字段(guanzhu_count): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: `... JOIN (主JOIN子查询-1.2) y2 ON y1.id = y2.id ...`
``` 
    - 检查训练数据: 
      - prompt 2/8, 错误都在"作为(表连接列)引用"
        - 检查的字段和 引用的片段 不一致. 检查训练数据, 并没有类似情况
    - 修正训练数据: 
      - 修正prompt 8的GT
      - 修正prompt 5的GT
      - 增强reward, 检查 "没有 被同级其他语句或父级各级语句 (...)" 中对...的约束, 需要按层次依次列出直到主语句, 并根据新的reward修复训练数据

再训练: 

![image2025-9-2 18:20:8.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-2%2018%3A20%3A8.png)

  - train/reward不高, 其中 刚才修改的Action3ReferenceValidationReward 一直有扣分, 需要在prompt进行规则强调
  - eval/reward 仍不高
  - 分析completion
    - 出现这样的描述: 

```
  - 动作2: 检查语句(主语句-1)的各查询字段: 根据优化规则, 不需要检查主语句, 直接检查各个子语句
  - 动作2: 检查语句(UNION第一部分-1.1)的各查询字段: (plot_id, variety_id, plot_name, variety_name, yield_lbs, productivity_level)
    - 动作3: 检查语句(UNION第一部分-1.1)的查询字段(plot_id): 被 同级其他语句或父级各级语句(UNION第一部分-1.1, UNION第二部分-1.2, UNION第三部分-1.3)作为(选择字段/表连接列/计算列/条件列/等)引用: SELECT pc.plot_id, pc.variety_id, f.plot_name, cv.variety_name, (pc.actual_yield_kg * 2.20462) AS yield_lbs, (CASE WHEN cv.average_yield_kg > 5000 THEN 'High' WHEN cv.average_yield_kg > 3000 THEN 'Medium' ELSE 'Low' END) AS productivity_level FROM (内层子查询-1.1.1) pc JOIN t_farmland f ON pc.plot_id = f.plot_id JOIN t_crop_variety cv ON pc.variety_id = cv.variety_id WHERE f.irrigation_method = 'Drip'
    - 动作3: 检查语句(UNION第一部分-1.1)的查询字段(variety_id): 被 同级其他语句或父级各级语句(UNION第一部分-1.1, UNION第二部分-1.2, UNION第三部分-1.3)作为(选择字段/表连接列/计算列/条件列/等)引用: SELECT pc.plot_id, pc.variety_id, f.plot_name, cv.variety_name, (pc.actual_yield_kg * 2.20462) AS yield_lbs, (CASE WHEN cv.average_yield_kg > 5000 THEN 'High' WHEN cv.average_yield_kg > 3000 THEN 'Medium' ELSE 'Low' END) AS productivity_level FROM (内层子查询-1.1.1) pc JOIN t_farmland f ON pc.plot_id = f.plot_id JOIN t_crop_variety cv ON pc.variety_id = cv.variety_id WHERE f.irrigation_method = 'Drip'
```

      - 需要检查"选择字段/表连接列/计算列/条件列/等", 对其扣分

      - 对主语句 和 UNION语句的父语句 的描述, 使用新的规则, 让其更明确

# 训练10: 多个修正

  - 在prompt中强调: 

```
- 对于"没有 被同级其他语句或父级各级语句({语句名列表})引用, 可以删除", 其中语句名列表需要列出你检查的各个语句名列表, 从父语句到父父语句直到主语句, 比如"AA-1.1.1,BB-1.1,主语句-1"
```

  - 在reward中增加检查: 需要检查"选择字段/表连接列/计算列/条件列/等", 对其扣分
    - 检查训练数据, 有一条数据因为出现了繁体, 导致此项扣分, 修复
  - 对主语句 和 UNION语句的父语句 的描述, 使用新的规则, 让其更明确, 修改训练数据

````
  - 对于集合操作所在的SQL（UNION、UNION ALL、INTERSECT、EXCEPT等）, 输出举例(其中X是一个UNION语句, Y是X的第一个子句): 
  	```
  	- 动作2: 检查语句(X)的各查询字段: (...), 该语句是一个集合操作(UNION)语句, 其查询字段与第一个子句相同, 直接分析第一个子句
  		- 动作2: 检查语句(Y)的各查询字段: (...)
  			- 动作3: 检查语句(Y)的查询字段(...): 
  	```

  替换"根据优化规则, 不需要检查主语句, 直接检查各个子语句", 对于最外层的动作3, 增加了一种形式
  - "检查语句({语句名})的查询字段({字段内容}): 这是最外层语句的查询字段, 会被应用使用, 不可删除"
````

出现了"All completions are overlong and truncated"

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30

(红色为基线, 橙色为本次训练)

![image2025-9-3 10:7:12.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-3%2010%3A7%3A12.png)

  - 各reward正常
  - eval/reward 一直不高, 且有下降
  - 分析completion
    - 最大reward达不到1.0: prompt 1-4
      - prompt 1: GT中的级联删除的逻辑, 无法训练出效果: 

```
- 动作3: 检查语句(子查询-1.1.1)的查询字段(soil_type): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(选择字段/排序字段)引用: (... FROM (SELECT ...) AS sub1 ORDER BY soil_type DESC)。但由于父语句(主FROM子语句-1.1)中的字段(soil_type)已被判断为可以删除, 且其排序操作(ORDER BY soil_type)对于外部查询(主语句-1)的结果没有影响, 是冗余的。因此, 该字段的引用可以被视为无效, 可以删除
```

      - prompt 2: 不同表的同名列无法区分
      - prompt 3: 不同表的同名列无法区分
      - prompt 4: 评估数据
        - 需要修正校验数据的GT, 否则F1的计算会有错误 (已经看到了满分推理, 但只得到了0.56分)
        - 低分的问题是不能正确识别 级联删除, 在提示词中给足够提示? 
    - 长期达不到1.0: prompt 13/14/15
      - prompt 13: 展开了 "UNION ALL"的父语句
      - prompt 14: 展开了 "UNION ALL"的父语句
      - prompt 15: GT可能有错: 
        - 这个s.chengshi是在GROUP BY中使用的, 需要保留. 需要增加相关数据.

```
- 动作3: 检查语句(IN子查询的JOIN子语句-1.6.1)的查询字段(s.chengshi): 没有 被同级其他语句或父级各级语句(主WHERE子查询-1.6, 主语句-1)引用, 可以删除
``` 

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.9, epoch=30

![image2025-9-3 10:9:47.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-3%2010%3A9%3A47.png)

  - 各reward正常
  - eval/reward 一直不高

### 修复验证数据的GT, 顺便测试一下DORA:

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30

### ![image2025-9-3 15:26:14.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-3%2015%3A26%3A14.png)

  - 训练reward曲线良好, 比lora的收敛效果好
  - 查询字段中出现引号, (与分割表名的逻辑冲突), 需要兼容: 

```
检查语句(主FROM子语句-1.1)的查询字段("months")
```

  - 验证数据中, 大部分问题是 不能识别级联删除 (随着DORA使用, 学习的方向不同)

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30

深红色为基线, 浅红色为本次训练

![image2025-9-2 10:7:40.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-2%2010%3A7%3A40.png)

  - train/reward增长形式正常
  - eval/reward中, Action3F1到0.8, 但Action2StructCompliance一直在扣分 (学不到这个知识)
  - 分析completion
    - 最大reward达不到1.0: prompt 1-8
      - prompt 7: 评估数据: 主要问题是将UNION的父语句展开分析了, 导致后续的分析偏差   

        - 需要一个更好的方式处理UNION的父语句?
      - prompt 1: 错误的分析: 
        - 需要检查"被同级其他语句或父级各级语句"引用中的列表

```
- 动作3: 检查语句(主SELECT第一部分-1.1)的查询字段(c.neirong): 没有 被同级其他语句或父级各级语句(主SELECT第二部分-1.2)引用, 可以删除
``` 
      - prompt 2: 错误的分析: 
        - 原因不详, m.machine_id确实是一个JOIN的键, 但这里JOIN的片段是错的, 而且也应当能删除

```
- 动作3: 检查语句(UNION第一部分-1.1.1)的查询字段(m.machine_id): 被 同级其他语句或父级各级语句(主语句-1, 子查询q-1.1, UNION第二部分-1.1.2)作为(表连接列)引用: `JOIN (子查询q-1.1) q ON p.line_id = q.production_line_id`
``` 
      - prompt 3: 
        - 删除了占位用的null

```
    - 动作3: 检查语句(UNION第三部分-1.3)的查询字段(null): 没有 被同级其他语句或父级各级语句(UNION ALL第一部分-1.1, UNION ALL第二部分-1.2)引用, 可以删除
    - 动作3: 检查语句(UNION第三部分-1.3)的查询字段(null): 没有 被同级其他语句或父级各级语句(UNION ALL第一部分-1.1, UNION ALL第二部分-1.2)引用, 可以删除
    - 动作3: 检查语句(UNION第三部分-1.3)的查询字段(null): 没有 被同级其他语句或父级各级语句(UNION ALL第一部分-1.1, UNION ALL第二部分-1.2)引用, 可以删除
``` 
      - prompt 4: 不能区分不同表的同名列
      - prompt 5: GT错误:
        - 这个排序应当不能删除

```
- 动作3: 检查语句(内层子查询-1.1.1)的查询字段(soil_type): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(排序列)引用: ORDER BY soil_type DESC。但由于主FROM子语句-1.1中的soil_type字段可以删除, 可以认为soil_type没有被有效引用, 可以删除
``` 
      - prompt 6: 不能区分不同表的同名列:

```
- 动作3: 检查语句(子查询nx-1.2)的查询字段(gongchang_id): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: `ns.gongchang_id = ng.id`
```

  
  

      - prompt 8: 分析错误, 原因不详
        - guanzhu_count在结构分析中被拆出一个子查询, 但实际上只是一个函数: "COUNT(DISTINCT g.beiguanzhu_id)". 尝试修正GT, 让结构拆解更合理

```
- 动作3: 检查语句(主JOIN子查询-1.2)的查询字段(guanzhu_count): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: `... JOIN (主JOIN子查询-1.2) y2 ON y1.id = y2.id ...`
``` 
    - 检查训练数据: 
      - prompt 2/8, 错误都在"作为(表连接列)引用"
        - 检查的字段和 引用的片段 不一致. 检查训练数据, 并没有类似情况
    - 修正训练数据: 
      - 修正prompt 8的GT
      - 修正prompt 5的GT
      - 增强reward, 检查 "没有 被同级其他语句或父级各级语句 (...)" 中对...的约束, 需要按层次依次列出直到主语句, 并根据新的reward修复训练数据

再训练: 

![image2025-9-2 18:20:8.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-2%2018%3A20%3A8.png)

  - train/reward不高, 其中 刚才修改的Action3ReferenceValidationReward 一直有扣分, 需要在prompt进行规则强调
  - eval/reward 仍不高
  - 分析completion
    - 出现这样的描述: 

```
  - 动作2: 检查语句(主语句-1)的各查询字段: 根据优化规则, 不需要检查主语句, 直接检查各个子语句
  - 动作2: 检查语句(UNION第一部分-1.1)的各查询字段: (plot_id, variety_id, plot_name, variety_name, yield_lbs, productivity_level)
    - 动作3: 检查语句(UNION第一部分-1.1)的查询字段(plot_id): 被 同级其他语句或父级各级语句(UNION第一部分-1.1, UNION第二部分-1.2, UNION第三部分-1.3)作为(选择字段/表连接列/计算列/条件列/等)引用: SELECT pc.plot_id, pc.variety_id, f.plot_name, cv.variety_name, (pc.actual_yield_kg * 2.20462) AS yield_lbs, (CASE WHEN cv.average_yield_kg > 5000 THEN 'High' WHEN cv.average_yield_kg > 3000 THEN 'Medium' ELSE 'Low' END) AS productivity_level FROM (内层子查询-1.1.1) pc JOIN t_farmland f ON pc.plot_id = f.plot_id JOIN t_crop_variety cv ON pc.variety_id = cv.variety_id WHERE f.irrigation_method = 'Drip'
    - 动作3: 检查语句(UNION第一部分-1.1)的查询字段(variety_id): 被 同级其他语句或父级各级语句(UNION第一部分-1.1, UNION第二部分-1.2, UNION第三部分-1.3)作为(选择字段/表连接列/计算列/条件列/等)引用: SELECT pc.plot_id, pc.variety_id, f.plot_name, cv.variety_name, (pc.actual_yield_kg * 2.20462) AS yield_lbs, (CASE WHEN cv.average_yield_kg > 5000 THEN 'High' WHEN cv.average_yield_kg > 3000 THEN 'Medium' ELSE 'Low' END) AS productivity_level FROM (内层子查询-1.1.1) pc JOIN t_farmland f ON pc.plot_id = f.plot_id JOIN t_crop_variety cv ON pc.variety_id = cv.variety_id WHERE f.irrigation_method = 'Drip'
```

      - 需要检查"选择字段/表连接列/计算列/条件列/等", 对其扣分

      - 对主语句 和 UNION语句的父语句 的描述, 使用新的规则, 让其更明确

# 训练10: 多个修正

  - 在prompt中强调: 

```
- 对于"没有 被同级其他语句或父级各级语句({语句名列表})引用, 可以删除", 其中语句名列表需要列出你检查的各个语句名列表, 从父语句到父父语句直到主语句, 比如"AA-1.1.1,BB-1.1,主语句-1"
```

  - 在reward中增加检查: 需要检查"选择字段/表连接列/计算列/条件列/等", 对其扣分
    - 检查训练数据, 有一条数据因为出现了繁体, 导致此项扣分, 修复
  - 对主语句 和 UNION语句的父语句 的描述, 使用新的规则, 让其更明确, 修改训练数据

````
  - 对于集合操作所在的SQL（UNION、UNION ALL、INTERSECT、EXCEPT等）, 输出举例(其中X是一个UNION语句, Y是X的第一个子句): 
  	```
  	- 动作2: 检查语句(X)的各查询字段: (...), 该语句是一个集合操作(UNION)语句, 其查询字段与第一个子句相同, 直接分析第一个子句
  		- 动作2: 检查语句(Y)的各查询字段: (...)
  			- 动作3: 检查语句(Y)的查询字段(...): 
  	```

  替换"根据优化规则, 不需要检查主语句, 直接检查各个子语句", 对于最外层的动作3, 增加了一种形式
  - "检查语句({语句名})的查询字段({字段内容}): 这是最外层语句的查询字段, 会被应用使用, 不可删除"
````

出现了"All completions are overlong and truncated"

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30

(红色为基线, 橙色为本次训练)

![image2025-9-3 10:7:12.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-3%2010%3A7%3A12.png)

  - 各reward正常
  - eval/reward 一直不高, 且有下降
  - 分析completion
    - 最大reward达不到1.0: prompt 1-4
      - prompt 1: GT中的级联删除的逻辑, 无法训练出效果: 

```
- 动作3: 检查语句(子查询-1.1.1)的查询字段(soil_type): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(选择字段/排序字段)引用: (... FROM (SELECT ...) AS sub1 ORDER BY soil_type DESC)。但由于父语句(主FROM子语句-1.1)中的字段(soil_type)已被判断为可以删除, 且其排序操作(ORDER BY soil_type)对于外部查询(主语句-1)的结果没有影响, 是冗余的。因此, 该字段的引用可以被视为无效, 可以删除
```

      - prompt 2: 不同表的同名列无法区分
      - prompt 3: 不同表的同名列无法区分
      - prompt 4: 评估数据
        - 需要修正校验数据的GT, 否则F1的计算会有错误 (已经看到了满分推理, 但只得到了0.56分)
        - 低分的问题是不能正确识别 级联删除, 在提示词中给足够提示? 
    - 长期达不到1.0: prompt 13/14/15
      - prompt 13: 展开了 "UNION ALL"的父语句
      - prompt 14: 展开了 "UNION ALL"的父语句
      - prompt 15: GT可能有错: 
        - 这个s.chengshi是在GROUP BY中使用的, 需要保留. 需要增加相关数据.

```
- 动作3: 检查语句(IN子查询的JOIN子语句-1.6.1)的查询字段(s.chengshi): 没有 被同级其他语句或父级各级语句(主WHERE子查询-1.6, 主语句-1)引用, 可以删除
``` 

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.9, epoch=30

![image2025-9-3 10:9:47.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-3%2010%3A9%3A47.png)

  - 各reward正常
  - eval/reward 一直不高

### 修复验证数据的GT, 顺便测试一下DORA:

### 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30

### ![image2025-9-3 15:26:14.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-3%2015%3A26%3A14.png)

  - 训练reward曲线良好, 比lora的收敛效果好
  - 查询字段中出现引号, (与分割表名的逻辑冲突), 需要兼容: 

```
检查语句(主FROM子语句-1.1)的查询字段("months")
```

  - 验证数据中, 大部分问题是 不能识别级联删除 (随着DORA使用, 学习的方向不同)

# 训练11: 增加对于级联删除的强调

  - 需要重新生成数据30 (结构分析有错误)
  - 数据23/26分析有误, 已修正
  - 加强其他数据中, 对于级联删除的描述: 
    - 举例:

```
    - 动作3: 检查语句(子查询v2-1.1.3)的查询字段(cost): 被 同级其他语句或父级各级语句(主FROM子语句-1.1)作为(选择字段)引用: `v2.cost`, 但引用处已经被标记成可删除, 当前字段已没有其他引用, 可以删除
``` 
  - 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.3, epoch=30
    - 红色为基线, 橙色为本次训练
    - ![image2025-9-4 10:8:46.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-4%2010%3A8%3A46.png)
    - train/Action3ReferenceValidationReward 一直有扣分, 且train/Action3F1Reward达不到极值
    - eval/Action3F1Reward 的趋势变好
    - 分析completion: 
      - prompt 1-9, 最大reward达不到1.0
        - prompt 1: GT错误
        - prompt 2: 对一个字段的引用判断, 会一直出错 (所有completion中没有正确): 
          - 认为是基模能力不足

```
- 动作3: 检查语句(折扣条件子查询-1.1.1)的查询字段(fx.fuwuming): 被 同级其他语句或父级各级语句(内层联合查询-1.1, 主语句-1)作为(选择字段)引用: `SELECT d.bianhao, f.baofeijine, fb.jibie, ...`, `SELECT d.bianhao, f.baofeijine, fb.jibie, ...`
``` 
        - prompt 3: 对一个字段的引用判断, 会一直出错 (所有completion中没有正确):

```
- 动作3: 检查语句(UNION第一部分-1.1.1)的查询字段(m.machine_id): 被 同级其他语句或父级各级语句(子查询q-1.1, 主语句-1)作为(表连接列)引用: (SELECT p.line_id = q.production_line_id FROM t_production_lines p JOIN (子查询q-1.1) q ON p.line_id = q.production_line_id)
```

        - prompt 4: 对一个字段的引用判断, 会一直出错 (所有completion中没有正确):

          - GT中: 被计算列引用. 但推理中认为没有引用

```
- 动作3: 检查语句(包裹子查询-1.1)的查询字段(package_id): 被 父级各级语句(主语句-1)作为(计算列)引用: (COUNT(p.package_id) AS package_count)
``` 
        - prompt 5: 无法判断级联删除
          - GT

```
- 动作3: 检查语句(内层FROM子语句-1.1.1)的查询字段(v2.manufacture_date): 被父级各级语句(主FROM子语句-1.1)作为(选择字段)引用: `SELECT v1.manufacture_date ...`, 但引用处已经被标记成可删除, 当前字段已没有其他引用, 可以删除
``` 
        - prompt 6: 不同表的同名列

```
- 动作3: 检查语句(诊断记录子查询-1.1)的查询字段(pd.department_id): 被 同级其他语句或父级各级语句(主语句-1)作为(条件列)引用: (AND d.department_id = 'DEP001')
``` 
        - prompt 7: 评估数据
          - 没有明确的"级联删除"相关的字样, 有一些应当被级联删除的列 没有处理

        - prompt 8: 不同表的同名列

```
- 动作3: 检查语句(子查询nx-1.2)的查询字段(gongchang_id): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: (ns.id, ns.shebei_mingcheng, nx.xiangmu_mingcheng, ng.gongchang_mingcheng)
```

        - prompt 9: 不能识别级联删除

```
- 动作3: 检查语句(子查询B-1.1.2)的查询字段(consultation_duration): 没有 被同级其他语句或父级各级语句(主FROM子语句-1.1, 主语句-1)引用, 可以删除
 
...
 
 - 动作3: 检查语句(内层子查询B-1.1.2.1)的查询字段(consultation_fee): 没有 被同级其他语句或父级各级语句(子查询B-1.1.2, 主FROM子语句-1.1, 主语句-1)引用, 可以删除
```

      - prompt 16, reward长期达不到1.0
    - 整体感受: 除了老问题 "不同表的同名列"判断有问题, 其他问题集中于 对于某一列的引用判断有误, 且基模型很固执, 没有给GRPO学习留下空间
  - 使用20条数据进行sft, 使用20条数据进行grpo, temperature=0.9, epoch=30
    - 橙色为上次训练(temperature=0.3), 红色为本次训练 (temperature=0.9)
    - ![image2025-9-4 14:24:46.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-4%2014%3A24%3A46.png)
    - train和eval的效果都更好
    - 并且train/Action3ReferenceValidationReward的扣分消失

观察completion (temperature=0.9的一次训练): 

  - 一共有3432个completion, 但只有33次包含"引用处"关键字 (级联删除的标准语句的关键字)
  - 检查训练数据, 7/20 数据有"引用处"关键字
  - 认为基模型对"级联删除"的概念理解不深, 没有给GRPO留下思考空间

# 训练12: 调整训练数据, 只使用级联删除相关的case

另一个思路: 考虑使用DPO?

只使用数据 1/2/3/4/5/12/14/21/22/23/26 , 一共11个数据

sft epoch=12, 在初期completion中, 即使对于训练数据本身, "引用处"关键字出现的仍然不多. 在sft后增加infer, 来看微调后的模型能力: 

```
CUDA_VISIBLE_DEVICES=0 swift infer  --adapters "$last_model_checkpoint"  --infer_backend vllm     --temperature 0.6     --max_new_tokens 6000  --val_dataset /data/huangyan/sql-ai-2/make_train_dataset.v3/val_data.jsonl  --max_batch_size 1 
``` 

vllm不支持dora, 先切换回lora

调整epoch为12/6/1, temperture=0.6, 对评估数据, 始终无法出现"引用处"

调整epoch为12, temperature=1.5, 对评估数据 (重复8次), 出现33个"引用处"

  - (jq -r '.response' /data/huangyan/sql-ai-2/train_logs/2025-09-04_16-22-10/train_output/v0-20250904-162230/checkpoint-12/infer_result/20250904-163033.jsonl | grep '引用处' | wc -l)

调整epoch为12, temperature=1.2, 对评估数据 (重复8次), 出现10个"引用处"

调整epoch为12, temperature=0.9, 对评估数据 (重复8次), 出现0个"引用处"

调整epoch为6, temperature=1.5, 对评估数据 (重复8次), 出现2个"引用处"

调整epoch为6, temperature=1.2, 对评估数据 (重复8次), 出现0个"引用处"

调整epoch为18, temperature=1.5, 对评估数据 (重复8次), 出现0个"引用处"

调整epoch为18, temperature=1.2, 对评估数据 (重复8次), 出现1个"引用处"

调整epoch为0, temperature=0.9, 对评估数据 (重复8次), 出现0个"引用处"

使用Qwen3-8B裸模型, temperature=1.2, 对评估数据 (重复8次), 出现0个"引用处"

使用Qwen3-14B裸模型, temperature=1.2, 对评估数据 (重复8次), 出现5个"引用处"

在提示词中强调 (你要特别注意判断判断"引用处已经被标记成可删除"这种形式, 这经常容易判断错误):

  - 使用Qwen3-14B裸模型, temperature=1.2, 对评估数据 (重复8次), 出现21个"引用处"
  - 使用Qwen3-14B裸模型, temperature=0.9, 对评估数据 (重复8次), 出现24个"引用处"
  - 使用Qwen3-8B裸模型, temperature=1.2, 对评估数据 (重复8次), 出现0个"引用处"
  - 调整epoch为12 (在训练提示词中没有强调), temperature=1.2, 对评估数据 (重复8次), 出现0个"引用处"

14B的理解会好很多, 修改训练提示词(增加强调), 然后从14B裸模型开始进行GRPO

  - 注意: 换回原来的训练数据

# 训练13: 14B裸模型直接开始进行GRPO, 换回原来的20条训练数据

检查训练初期: 评估数据的completion, 频繁出现了级联删除的逻辑. 

使用20条数据进行grpo, temperature=0.9, epoch=30

  - 发现一条数据GT错误
  - ![image2025-9-5 10:26:20.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-5%2010%3A26%3A20.png)
  - eval/F1 只到 0.77
  - 检查completion: 
    - 最大reward不能到1.0: prompt 1-5
      - prompt 1: GT数据错误, 在下次训练中修正
      - prompt 2: 
        - 随着训练进行, UNION父语句被错误展开
        - 始终有一个错误判断:

```
- 动作3: 检查语句(UNION第一部分-1.1.1)的查询字段(m.machine_id): 被 同级其他语句或父级各级语句(子查询q-1.1, 主语句-1)作为(表连接列)引用: (JOIN ... ON p.line_id = q.production_line_id), 不可删除
```

      - prompt 3: 评估数据: 
        - 已经能正确的进行级联删除
        - 发现评估数据的GT错误
        - 对于选择列中出现连接列, 会出现错误的判断: 

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.months): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: (ON v1.months = t1.months), 不可删除
```

      - prompt 4:
        - GT错误, 少了一个级联删除

```
- 动作3: 检查语句(内层子查询A-1.1.1.1)的查询字段(consultation_duration): 被 同级其他语句或父级各级语句(子查询A-1.1.1, 主FROM子语句-1.1)作为(选择字段)引用: (SELECT consultation_duration ...), 但引用处已经被标记成可删除, 当前字段已没有其他引用, 可以删除
```

      - prompt 5: 
        - 不能正确判断计算列: 

```
GT: 
- 动作3: 检查语句(包裹子查询-1.1)的查询字段(package_id): 被 父级各级语句(主语句-1)作为(计算列)引用: (COUNT(p.package_id) AS package_count)
 
推理: 
- 动作3: 检查语句(包裹子查询-1.1)的查询字段(package_id): 没有 被同级其他语句或父级各级语句(主语句-1)引用, 可以删除
``` 
    - reward长期达不到1.0: prompt 11
      - UNION父语句被展开, 导致删除多了一个, 问题不大
  - 使用了14B模型后, 不同表的同名列的错误判断 消失了

使用30条数据进行grpo, temperature=0.9, epoch=30

  - 修正上面的数据错误
  - (红色为上次训练, 蓝色为本次训练)
  - ![image2025-9-5 11:0:34.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-5%2011%3A0%3A34.png)
  - 训练曲线正常, 但评估曲线崩溃
  - 分析completion: 
    - 最大reward不能到1.0: prompt 1-7
      - prompt 1: 同上一次训练的2
      - prompt 2: GT中的结构分析错误, 将表作为了单独的子查询, 导致分析时匹配不上: 

```
SELECT patient_id, consultation_date FROM (SELECT patient_id, consultation_date, doctor_id, CASE WHEN treatment IS NOT NULL THEN 'Treated' ELSE 'Pending' END AS treatment_status FROM (SELECT pc.patient_id, pc.consultation_date, pc.doctor_id, pc.treatment FROM t_patient_consultation pc INNER JOIN t_doctor_specialty ds ON pc.doctor_id = ds.doctor_id WHERE ds.specialty_name = 'Cardiology') AS subquery1) AS subquery2 ORDER BY consultation_date, treatment_status;
```

      - prompt 3: 对条件列的引用判断错误: 

```
- 动作3: 检查语句(诊断记录子查询-1.1)的查询字段(pd.department_id): 被 同级其他语句或父级各级语句(主语句-1)作为(条件列)引用: (AND d.department_id = 'DEP001'), 不可删除
```

      - prompt 4: 评估数据
        - 随着训练进行, 级联删除的正确操作越来越少
      - prompt 5: 同上一次训练的4
      - prompt 6: 同上一次训练的5
      - prompt 7: 
        - 出现了对条件列的判断错误

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(chanpin_jiezhi): 被 同级其他语句或父级各级语句(主语句-1)作为(条件列)引用: (chanpin_jiezhi > '2023-01-01')
``` 
    - reward长期达不到1.0: prompt 10/27/28
      - prompt 10: 出现了对条件列的判断错误, 以及对having条件的判断错误: 

```
    - 动作3: 检查语句(IN子查询的JOIN子语句-1.6.1)的查询字段(s.chengshi): 被 IN子查询的JOIN子语句-1.6.1 作为(条件列)引用: (WHERE s.chengshi = '北京市'), 不可删除
    - 动作3: 检查语句(IN子查询的JOIN子语句-1.6.1)的查询字段(COUNT(sc.yonghu_id) AS chengyuan_count): 被 主WHERE子查询-1.6 作为(条件列)引用: (HAVING COUNT(sc.yonghu_id) > 100), 不可删除
```

      - prompt 27: 
        - 对外部条件列引用内部选择列, 出现了错误的判断: 

```
- 动作3: 检查语句(主FROM子语句-1.1.1.1)的查询字段(shebei_zhuangtai): 没有 被同级其他语句或父级各级语句(主FROM子语句-1.1.1, 主FROM子语句-1.1, 主语句-1)引用, 可以删除
``` 
      - prompt 28: 展开了UNION父语句, 导致多了一个删除

# 训练14: 8B模型使用DPO来增强级联删除的能力

TODO: /data/huangyan/llm_train_data_mgr/backend/data/exports/dataset_20250904_190214.dpo.jsonl

commit: ...

实验: 8次eval评估

  - sft后: 出现16次 "引用处"
  - dpo (rpo_alpha=1.0, epoch=12, LR=1e-5) 后: 出现29次 "引用处"
    - 出现了更多的 级联删除 操作, 但逻辑是错误的: 出现了比较极端的情况, 全部都删除, 或者全部不删除
  - dpo (rpo_alpha=1.0, epoch=12, LR=5e-6) 后: 出现16次 "引用处"
  - dpo (rpo_alpha=1.0, epoch=12, LR=1e-6) 后: 出现16次 "引用处"
  - dpo (rpo_alpha=1.0, epoch=20, LR=1e-5) 后: 出现32次 "引用处"
    - 出现了更多的 级联删除 操作, 但逻辑是错误的: 出现了比较极端的情况, 全部都删除, 或者全部不删除
  - dpo (rpo_alpha=1.0, epoch=12, LR=5e-5) 后: 出现79次 "引用处"
    - rewards/accuracies = 1.0, 对训练数据已经足够好, 但评估数据的accuracies变化不大
    - 出现了更多的 级联删除 操作, 但逻辑是错误的: 出现了比较极端的情况, 全部都删除, 或者全部不删除

现在的DPO使用的对比数据, 都是倾向于更多的"级联删除"操作, 需要构造一些反向的数据

TODO

# 训练15: 14B裸模型直接开始进行GRPO, 修复数据问题

修复数据:

  - 修复训练数据
  - 修复数据14: 少了一个级联删除
  - 修复数据23: 重新进行结构分析, 不让表作为单独的子查询结构

进行训练: (使用30条数据进行grpo, temperature=0.9, epoch=20, dora): (蓝色是训练13, 绿色曲线为本次训练)

![image2025-9-5 16:16:36.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-5%2016%3A16%3A36.png)

  - 因为修正了评估数据的GT, 所以eval/reward变高, 但训练趋势是类似的 (比起上一次训练, 修正的数据不多)
  - 尝试在当前的checkpoint上, 进行续训
    - 出现了"[WARNING:swift] There are still std=0 groups present after 6 retries.", 导致续训时间变长 (将近两倍)
    - 续训曲线: (白色曲线为续训曲线)
    - ![image2025-9-5 23:29:46.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-5%2023%3A29%3A46.png)
    - 检查completion:
      - 最大reward达不到1.0: prompt 1-8
        - prompt 1: 子语句的order by列, 对父语句没影响, 但被判定为不能删除

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(soil_type): 被 父级各级语句(主FROM子语句-1.1)作为(排序列)引用: (ORDER BY soil_type DESC), 不可删除
```

        - prompt 2: 对连接列的判断错误: 

```
- 动作3: 检查语句(UNION第二部分-1.1.2)的查询字段(m.machine_id): 被同级其他语句或父级各级语句(子查询q-1.1, 主语句-1)作为(表连接列)引用: (ON p.line_id = q.production_line_id), 不可删除
```

        - prompt 3: 多删除了外查询引用的条件列: 

```
- 动作3: 检查语句(主FROM子语句-1.1.1)的查询字段(shebei_zhuangtai): 没有 被同级其他语句或父级各级语句(主FROM子语句-1.1, 主语句-1)引用, 可以删除
```

        - prompt 4: 展开了UNION父查询, 导致多了一个删除
        - prompt 5: 多删除了外查询引用的条件列: 

```
- 动作3: 检查语句(包裹子查询-1.1)的查询字段(package_id): 没有 被同级其他语句或父级各级语句(主语句-1)引用, 可以删除
```

        - prompt 6: 评估数据: 
          - 选择列 也是 连接列, 被错误地判断成不能删除: 

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.months): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: (ON v1.months = t1.months, ON v1.months = v2.state_date, ON v1.months = v3.months), 不可删除
```

        - prompt 7: 展开了UNION父查询, 导致多了一个删除
        - prompt 8: GT中结构错误, 将一个CASE子句作为了单独的结构: 

```
select a.chanpin_id, a.chanpin_mingcheng, b.xiaoshou_jine, b.xiaoshou_shuliang from (select chanpin_id, chanpin_mingcheng, chanpin_leixing, chanpin_jiezhi, chanpin_chandi, chanpin_jiazhi, chanpin_shuliang, chanpin_zhuangtai, chanpin_beizhu, chanpin_shijian, chanpin_quyu, chanpin_yongtu from nengyuan_chanpin where chanpin_jiezhi > '2023-01-01') a join nengyuan_xiaoshou b on a.chanpin_id = b.chanpin_id and a.chanpin_quyu = b.xiaoshou_quyu where b.xiaoshou_jine > case when a.chanpin_jiazhi > 10000 then 500 else 100 end;
```

      - reward长期达不到1.0: prompt 9
        - 对having条件和普通条件的引用列 错误地判断为不可删除: 

```
    - 动作3: 检查语句(IN子查询的JOIN子语句-1.6.1)的查询字段(s.chengshi): 被 IN子查询的JOIN子语句-1.6.1 作为(条件列)引用: (where s.chengshi = '北京市'), 不可删除

    - 动作3: 检查语句(IN子查询的JOIN子语句-1.6.1)的查询字段(COUNT(sc.yonghu_id) AS chengyuan_count): 被 主WHERE子查询-1.6 作为(条件列)引用: (having count(sc.yonghu_id) > 100), 不可删除
```

    - 使用temperature=1.5进行续训: 
      - (绿色曲线为第一阶段, 黄色曲线为temperature=0.9的第二次训练, 橙色曲线为temperature=1.5的第二次训练)
      - ![image2025-9-6 10:44:5.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-6%2010%3A44%3A5.png)
      - train/F1 比temperature 0.9略低
      - eval/F1 比temperature 0.9更好
      - 分析completion:
        - 评估数据: 与temperature=0.9相比, 最优解都是0.88, 问题一样. 区别在于本次 的解比较文档, 都是0.88, 而不波动. 也就是说temperature=1.5找到了一个更稳定的方向

检查数据:

  - 检查训练数据中, "UNION父语句被错误展开"的情况: 
    - 并没有出现UNION语句被展开的情况
    - 评估裸模型的能力, 考虑是否增加sft?
  - 检查训练数据中, 对"连接列"的判断 (之前的错误是 在选择中出现了连接列, 误判成了 "不可删除", 举例: select a.a from ... where a.a = b.b, a.a是可以删除的)
    - 没发现问题.
  - TODO: 检查训练数据中, 对"计算列"和"条件列"和"having条件"的判断

思考: 前两次训练都是从裸模型进行GRPO, 所以训练数据的GT并不会加入模型能力 (只影响reward计算时的参考项) 

  - 考虑增加sft或者DPO? 

# 训练16: 对于GRPO后, 基模型完全无法探索的case, 进行特别修正

目标: 修复评估数据的错误: "选择列 也是 连接列, 被错误地判断成不能删除"

想法1:

  1. 从训练数据中找到 GT删除的选择列 是 同级的连接列.
  2. 生成一些劣化数据, 将这些列判断成不可删除, 使用sft, 混入一些劣化数据
  3. 检验基模型, 可以概率的输出正确答案和劣化答案
  4. 再使用GRPO进行训练
  5. 观察评估数据

想法2: 在GRPO模型后, 使用DPO (使用想法1的劣化数据), 观察效果

训练数据中, 23/17/16/11 四个数据 被删除的列 跟 连接列 是相同的, 生成劣化数据.

实验1:

  - GRPO, 30条数据, epoch=5
  - DPO, 使用23/17/16/11的劣化数据构成DPO, epoch=6
    - "rewards/accuracies" 始终为 1.0
  - 没有实际效果

实验2: 

  - 使用劣化数据进行sft
  - 用infer进行检验, 部分数据仍然保持了劣化前的状态, 并没有学到劣化的特点
  - 调整参数: lr=5e-4, epoch=12, 完全过拟合, 可以学到劣化数据. 需要找到平衡点
  - 使用lr=5e-4, epoch=6
  - GRPO效果不好:
    - ![image2025-9-7 18:41:56.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-7%2018%3A41%3A56.png)

实验3: 

  - 将提示词中的"引用", 改成"直接使用", 并做出说明

```
- "直接使用"指的是 当前语句直接使用了该字段, 但不穿透子语句
```

  
  

    - 其中"穿透子语句"这个说法, 是让Qwen3-8B对错误作出分析, 由人工引导, 而产生出来的概念
  - 使用infer来对Qwen3-8B的基模能力进行观察 (temperature=1.0), 2/8能做出正确的判断 (修改提示词之前 0/8 能做出正确判断)
    - 但对于Qwen3-14B, 无法得到正确判断
  - 修改提示词, 以在Qwen3-14B上获得3/8的正确的判断: 
    - 增加 "(而不是删除表中的该字段)" 以及 "({详细说明})":

```
  - "检查语句({语句名})的查询字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 只删除查询字段(而不是删除表中的该字段)不会影响 被引用处({选择字段/表连接列/计算列/条件列/等})的使用({详细说明}), 可以删除"
  - "检查语句({语句名})的查询字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 只删除查询字段(而不是删除表中的该字段)会导致被引用处({选择字段/表连接列/计算列/条件列/等})无法引用该字段({详细说明}), 不可删除" 
``` 
    - commit: a1640d34978ee9ab16379715f567c08fa7e13bd1
    - 进行GRPO训练: LR=4e-5:
      - ![image2025-9-8 17:58:7.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-8%2017%3A58%3A7.png)
      - 需要分析completion
        - 最高reward达不到1.0: prompt 1-6:
          - prompt 1: 无法进行级联删除
            - 数据14

```
- 动作3: 检查语句(子查询A-1.1.1)的查询字段(consultation_duration): 被 主FROM子语句-1.1 作为(选择字段)引用: (SELECT consultation_duration ...), 只删除查询字段(而不是删除表中的该字段)会导致被引用处(选择字段)无法引用该字段(主FROM子语句-1.1需要该字段作为输入), 不可删除
``` 
          - prompt 2: 无法删除join列
            - 数据16

```
- 动作3: 检查语句(子查询q-1.1)的查询字段(m.machine_id): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: (ON p.line_id = q.production_line_id), 只删除查询字段(而不是删除表中的该字段)会导致被引用处(表连接列)无法引用该字段(该字段用于连接t_production_lines表和子查询q-1.1的结果), 不可删除
``` 
          - prompt 3: 无法删除条件列
            - 数据20

```
- 动作3: 检查语句(诊断记录子查询-1.1)的查询字段(pd.department_id): 被 主语句-1 作为(条件列)引用: (AND d.department_id = 'DEP001'), 只删除查询字段(而不是删除表中的该字段)会导致被引用处(条件列)无法引用该字段(该字段用于筛选医生所属科室), 不可删除
``` 
          - prompt 4: 错误删除了外层条件引用的列: 
            - 数据13

```
GT:
- 动作3: 检查语句(包裹子查询-1.1)的查询字段(package_id): 被 父级各级语句(主语句-1)作为(计算列)引用: (COUNT(p.package_id) AS package_count)
 
推理: 
- 动作3: 检查语句(包裹子查询-1.1)的查询字段(package_id): 没有 被同级其他语句或父级各级语句(主语句-1)引用, 可以删除
``` 
          - prompt 5: 不能删除条件列: 
            - 数据24

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(chanpin_jiezhi): 被 父级各级语句(主FROM子语句-1.1)作为(条件列)引用: (chanpin_jiezhi > '2023-01-01'), 只删除查询字段(而不是删除表中的该字段)会导致被引用处(条件列)无法引用该字段(该字段用于过滤数据), 不可删除
``` 
          - prompt 6: 出现了错误的逻辑: 

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.months): 被 父级语句(主语句-1)作为(选择字段)引用: (SELECT AVG(A44_T_1_."fd_roi") ...), 但引用处已经被标记成不可删除, 当前字段已没有其他引用, 不可删除
```

        - reward长期达不到1.0: prompt 7/27:
          - prompt 7: 展开了union父查询, 多了一个删除
          - prompt 27: GT 表结构错误, 需要重新生成GT结构
    - 用LR=2e-5再跑一次:
      - (蓝色曲线为本次训练, 橙色曲线为上次训练(LR=4e-5))
      - ![image2025-9-8 22:15:8.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-8%2022%3A15%3A8.png)
      - 本次训练在训练集上表现更好, 但在校验集上表现更差
      - 在校验集上, 发现Action3ReferenceValidation 与 F1 呈相反趋势, 考虑去掉Action3ReferenceValidation的训练进行尝试

# 训练17: 验证基模型能力, 并去掉Action3ReferenceValidation进行训练

进行几个调整: 

  - 对于之前的几个问题, 在基模型14B上进行能力验证: 
    - 无法进行级联删除: 数据14
      - 裸模型可以删除
    - 无法删除join列: 数据16
      - 裸模型可以删除
    - 无法删除条件列: 数据20/24
      - 数据20, 裸模型可以删除, 但概率低
      - 数据24, 裸模型不能正确删除
    - 错误删除了外层条件引用的列: 数据13
      - 裸模型无法正确判断, 仍然会删除该列
    - 校验数据 (之前已经验证过其在v1.month字段上的删除能力)
  - ~~重新生成数据27的GT~~
  - ~~去掉Action3ReferenceValidation进行训练~~

修改提示词, 引入两个修改: 

  - 动作3的提示词修改为: 

```
动作3:
- 动作字样: 
  - "检查语句({语句名})的输出字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 删除该输出字段 不会影响 被引用处的使用({详细说明}), 可以删除"
  - "检查语句({语句名})的输出字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 删除该输出字段 会导致被引用处 无法引用该字段({详细说明}), 不可删除" 
  - "检查语句({语句名})的输出字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 被引用处已经被标记成可删除, 当前输出字段已没有其他引用, 可以删除"
  - "检查语句({语句名})的输出字段({字段内容}): 只被 当前语句作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 删除该输出字段 不会影响 被引用处的使用({详细说明}), 可以删除" 
  - "检查语句({语句名})的输出字段({字段内容}): 这是最外层语句的输出字段, 会被应用使用, 不可删除"
  - "检查语句({语句名})的输出字段({字段内容}): 没有 被同级其他语句或父级各级语句({语句名列表})引用, 可以删除"
- 动作说明: 
  - 必须作为动作2的子动作。
  - 遗漏语句或字段会扣分
  - 对于集合操作中的子查询，还需要考虑是否被集合操作的其他子查询所依赖（如UNION操作要求各子查询的字段数量和类型一致）
  - 注意: 如果字段是可以删除的, 那么必须使用"可以删除"这个字样来描述
  - 对于"没有 被同级其他语句或父级各级语句({语句名列表})引用, 可以删除", 其中语句名列表需要列出你检查的各个语句名列表, 列表顺序从父语句到祖先语句直到主语句, 比如"AA-1.1.1,BB-1.1,主语句-1"
  - 对于列出相关SQL片段, 需要保证片段是来自于原SQL的一部分, 并且片段足够简略, 可以使用省略号来突出相关性, 比如`... FROM TABLE WHERE ... AND 0 = 1`
  - 注意判断"只被当前语句引用"这种情况, 这经常容易判断错误
  - 注意判断"引用处已经被标记成可删除"这种情况, 这经常容易判断错误
```

    - 将所有查询字段改为输出字段
  - 只使用F1作为奖励  
  

尝试GRPO: 

  - ![image2025-9-9 9:58:35.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-9%209%3A58%3A35.png)
  - eval/reward很低, 但train/reward可以收敛
  - 分析completion
    - 最大reward达不到1.0: prompt 1-5
      - prompt 1: 分析是正确的, 但表名.列名匹配不上: 

```
GT: 
- 动作3: 检查语句(诊断记录子查询-1.1)的输出字段(pd.diagnose_time): 没有 被父级各级语句(主语句-1)引用, 可以删除
 
推理: 
- 动作3: 检查语句(诊断记录子查询-1.1)的输出字段(diagnose_time): 没有 被同级其他语句或父级各级语句(主语句-1)引用, 可以删除
```

      - prompt 2: 
        - 少了对中间一层的详细分析
        - 不能进行级联删除
      - prompt 3:   

        - 不能正确删除连接列

```
- 动作3: 检查语句(UNION第一部分-1.1.1)的输出字段(m.machine_id): 被 同级其他语句或父级各级语句(UNION第二部分-1.1.2, 子查询q-1.1, 主语句-1)作为(表连接列)引用: (JOIN ... ON m.machine_id = q.machine_id), 删除该输出字段 会导致被引用处 无法引用该字段(连接条件将失效), 不可删除
```

  
  

      - prompt 4: 评估数据: 有两类主要错误: 
        - 不能进行级联删除
        - 不能正确删除连接列
        - 表名.列名匹配不上
      - prompt 5: 错误删除了被父节点引用的条件列
      - 之前出现的不能删除条件列的问题 消失
    - 没有reward长期达不到1.0的case

这次训练, 是通过修改提示词, 让基模型遵循某种形式. 并通过简化reward, 让模型自由发挥. 碰到的问题是有一些方向(删除连接列, 级联删除) 在基模型能力中, 并没有得到发挥

# 训练18: 调整提示词, 只使用F1作为reward

调整提示词: 

```
动作3:
- 动作字样: 
  - "检查语句({语句名})的输出字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 删除该输出字段 不会影响 被引用处的使用({详细说明}), 可以删除"
  - "检查语句({语句名})的输出字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 删除该输出字段 会导致被引用处 无法引用该字段({详细说明}), 不可删除" 
  - "检查语句({语句名})的输出字段({字段内容}): 被 同级其他语句或父级各级语句({语句名列表})作为({选择字段/表连接列/计算列/条件列/等})引用: ({被引用的片段}), 被引用处已经被标记成可删除, 当前输出字段已没有其他引用, 可以删除"
  - "检查语句({语句名})的输出字段({字段内容}): 虽然未被父级语句引用, 但其源列在当前语句的({ON条件/WHERE条件等})中被引用: ({被引用的片段}), 且该输出字段是多余的, 可以删除"
  - "检查语句({语句名})的输出字段({字段内容}): 这是最外层语句的输出字段, 会被应用使用, 不可删除"
  - "检查语句({语句名})的输出字段({字段内容}): 虽然未被父级语句引用, 但为了与集合操作({UNION/INTERSECT/EXCEPT})中的其他子查询({语句名列表})在列数和类型上保持一致, 不可删除
  - "检查语句({语句名})的输出字段({字段内容}): 未被父级语句引用, 其源列在当前语句的其他部分也没有被引用, 可以删除"
- 动作说明: 
  - 必须作为动作2的子动作。
  - 遗漏语句或字段会扣分
  - 对于集合操作中的子查询，还需要考虑是否被集合操作的其他子查询所依赖（如UNION操作要求各子查询的字段数量和类型一致）
  - 注意: 如果字段是可以删除的, 那么必须使用"可以删除"这个字样来描述
  - 对于列出相关SQL片段, 需要保证片段是来自于原SQL的一部分, 并且片段足够简略, 可以使用省略号来突出相关性, 比如`... FROM TABLE WHERE ... AND 0 = 1`
  - 注意：判断一个输出字段是否被引用时，仅考虑它是否被父级查询、同级的其他子查询或集合操作所使用。该字段的源列在当前语句内部的WHERE、ON、GROUP BY等子句中的使用，不作为保留该输出字段的依据。
``` 

并将"选择字段"修改成"输出字段"

并改造reward, 支持"表名.列名匹配"

使用30条数据, 先sft(epoch=2) 再GRPO (epoch=20, 只使用F1): 

![image2025-9-9 19:1:25.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-9%2019%3A1%3A25.png)

未训练完, 人为中断. 分析completion: 

  - 评估数据: 没有出现级联删除的分析, 而级联删除的字段的分析是填写了其他原因

考虑换一个方式: 只使用"级联删除"相关的数据进行微调, 然后进行GRPO

  - 注意调整微调参数, 需要在微调后对校验数据进行验证, 其需要出现"级联删除"的分析步骤, 才能继续进行GRPO
  - sft参数: LR=1e-4, epoch=20 (才能出现"级联删除"步骤)
  - GRPO结果: 
    - step=10时, 校验数据的reward已经有的能达到1.0, 并且出现了"级联删除"的分析步骤
    - RUNNING

# 训练19: 使用chord

ms-swift 从 975b86b6cf7883fc23184f2cf8dfc9039507c65a 升级到 4a35605d9c56f21d33bcb91385b9a00653fe87ad

参数: 

```
"--chord_sft_per_device_train_batch_size 1 \
    --chord_sft_dataset $1 \
    --chord_enable_phi_function false \
    --chord_mu_warmup_steps 0 \
    --chord_mu_decay_steps $2 \
    --chord_mu_peak 0.9 \
    --chord_mu_valley 0.05"

``` 

chord_enable_phi_function=false, epoch=20, chord step=200, 总step=240. GAE=8. 数据: 

  - 因为脚本错误, chord sft使用的数据集是 11条与"连接列"相关的数据
  - grpo数据集是完整的30条数据集

效果: 训练需要9.5小时, 比起纯GRPO的6.5小时, 多了50%

(橙色为 sft+grpo训练, 没跑完. 蓝色为本次训练)

![image2025-9-10 10:8:31.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-10%2010%3A8%3A31.png)

  - 相比之下, 本次训练的收敛会更晚, 且eval/reward上会略低 (但比单纯的grpo好很多, 对比sft+grpo的方案略弱)
  - 另外, 没有启用chord_enable_phi_function参数
  - 分析completion: 
    - 评估数据: 
      - 已经出现了"级联删除"步骤, 也就是说chord的模仿功能生效
      - 最高得分, 遗漏了一个join列的删除 (之前的老问题):

```
- 动作3: 检查语句(主FROM子语句-1.1)的输出字段(months): 被 同级其他语句或父级各级语句(主语句-1)作为(选择字段)引用: (SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" FROM ...), 且其源列在当前语句的其他部分也没有被引用, 不可删除
```

    - 最高reward达不到1.0: prompt 1:
      - prompt 1: 不能正确删除join列: 

```
- 动作3: 检查语句(UNION第一部分-1.1.1)的输出字段(machine_id): 被 同级其他语句或父级各级语句(主语句-1)作为(表连接列)引用: (p.line_id = q.production_line_id), 删除该输出字段 会导致被引用处 无法引用该字段(父查询需要该字段与p.line_id进行关联), 不可删除
```

    - reward长期达不到1.0: prompt 23/29
      - prompt 23: 错误删除了被计算引用的列: 

```
GT: 
- 动作3: 检查语句(包裹子查询-1.1)的输出字段(package_id): 被 同级其他语句或父级各级语句(主语句-1)作为(计算列)引用: (COUNT(p.package_id) AS package_count), 删除该输出字段 会导致被引用处 无法引用该字段(COUNT聚合函数将失败), 不可删除
```

      - prompt 29: 无法删除条件列

```
- 动作3: 检查语句(诊断记录子查询-1.1)的输出字段(department_id): 被 同级其他语句或父级各级语句(主语句-1)作为(条件列)引用: (AND d.department_id = 'DEP001'), 删除该输出字段 会导致被引用处 无法引用该字段(AND d.department_id = 'DEP001' 会失效), 不可删除
```

    - 总体感觉: 
      - 出现问题的case, 在训练初期都能出现正确结果, 随着训练进行, 变得往错误的方向固执, 也就是说类似的数据少, 导致对改进方向判断有误

尝试续训, 使用连接列相关的新的数据 (2条 * 重复4次): epoch=4, chord

  - ![image2025-9-10 16:0:48.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-10%2016%3A0%3A48.png)
  - 红色曲线为续训, 蓝色曲线为初训
  - eval/reward增长不多, 但: 
    - 训练初期, eval中的 应当删除的join列, 完全不能删除
    - 训练中后期, eval中的 应当删除的join列, 倾向于被删除
  - 说明: 对于基模不具备的能力, chord可以将能力添加进去 (因为有sft阶段)

跑几个续训进行对比: 

  - chord_enable_phi_function=true, epoch=4, chord step=128, 总step=128. 2个跟join相关的新数据 * 4次重复. GAE=1.
    - ![image2025-9-11 10:21:59.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-11%2010%3A21%3A59.png)
    - 运行两次, eval/reward会有区别??
  - chord_enable_phi_function=true, epoch=4, chord step=128, 总step=480. 32个完整数据, GAE=1
    - ![image2025-9-11 10:23:9.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-11%2010%3A23%3A9.png)
    - 分析completion: 
      - eval在200步左右, 学到了"级联删除"这个操作. 
      - eval在最后, 出现了满分case(并且分析步骤是完全正确的), 一些扣分case主要是过度使用"级联删除", 错误删除了其他列

发现一个问题: 进行之前chrod训练时, grpo数据集和给cord的sft数据集不一致, 需要重新进行chord全数据训练

# 训练20: 使用chord, 重新训练

chord_enable_phi_function=true, epoch=8, chord step=960, 总step=960. 32个完整数据. GAE=1. LR=2e-5

![image2025-9-11 10:39:7.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-11%2010%3A39%3A7.png)

  - 效果很差
  - 与之前效果比较好的方案进行对比, 之前的方案: 
    - 初训:

chord_enable_phi_function=false, epoch=20, chord step=200, 总step=240. GAE=8. 数据: 

      - 因为脚本错误, chord sft使用的数据集是 11条与"连接列"相关的数据
      - grpo数据集是完整的30条数据集

续训:

chord_enable_phi_function=true, epoch=4, chord step=128, 总step=480. 32个完整数据, GAE=1

    - 认为在之前的方案中, 初训找到了局部最优解, 而续训是在局部最优解中精调, 跳出了局部, 提高了结论的泛化性 (让eval学到了 "级联删除"的知识)

调大LR=4e-5, 让初训能快速收敛:

  - 第一段训练: 导致了OOM, 直到epoch=40
    - 调大 vllm_gpu_memory_utilization 从 0.5 到 0.6
    - 其中有5个case, 在下次训练中, 已经全部满分, 导致模型无法找到差异
  - 第二段训练 (第1次): 以epoch=40为base, 进行续训
    - ![image2025-9-11 15:5:39.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-11%2015%3A5%3A39.png)
    - 蓝色为初训, 红色为本阶段续训
    - frac_reward_zero_std已经升的很高, 准备以此为基模, 调整GAS=1, 来模拟之前的训练
      - 发现reward完全为1, 训练不到任何东西
  - 重做第二段训练 (第2次): 以epoch=40为base, 调整GAS=1
    - ![image2025-9-12 12:56:6.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-12%2012%3A56%3A6.png)
    - 训练效果很好
    - 对于评估数据, step=250以后, 就在0.94 和 1.0 之间随机, 其中0.94是缺少了对join列的删除, 1.0是完全正确(且分析正确)
    - 分析completion: 
      - prompt 1: reward没有达到1.0, 且没有变化
        - 错误地删除了被父查询条件函数引用的列

```
GT: 
- 动作3: 检查语句(包裹子查询-1.1)的输出字段(package_id): 被 同级其他语句或父级各级语句(主语句-1)作为(计算列)引用: (COUNT(p.package_id) AS package_count), 删除该输出字段 会导致被引用处 无法引用该字段(COUNT聚合函数将失败), 不可删除
```

      - prompt 13: reward没有达到1.0, 且评分下降很厉害
        - GT中的结构分析错误, 将表名作为了单独的结构, 需要重新生成GT
  - 重做第二段训练 (第3次): 以epoch=40为base, 调整GAS=1, chord_enable_phi_function=true
    - 绿色曲线为本次续联, 红色曲线为chord_enable_phi_function=false的实验
    - ![image2025-9-12 12:59:39.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-12%2012%3A59%3A39.png)
    - 训练收敛稍慢, 但eval/reward表现不好
    - 验证completion, 确实会遗漏删除项

# 评估模型 "重做第二段训练 (第2次)"

使用孙健提供的122条"较难"的case: [rule_0001_hard_case_from_sj.jsonl](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/rule_0001_hard_case_from_sj.jsonl)

一共错误45条case: [filtered_evaluation_results.jsonl](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/filtered_evaluation_results.jsonl)

需要逐一分析:

  - id=14: GT对`eq.equipment_type`的判断错误
  - id=20: 窗口函数相关列, 不能正确判断
  - id=42: GT错误判断: `dx.shangpin_id` **不可删除**，因为它参与了 `GROUP BY` 
  - id=48: 对多表join的列的引用, 引用时没有使用表名, 识别错误
  - id=64: 对计算列和条件列引用判断错误, 少删除了
  - id=68: 与计算列有关的级联删除
  - id=72: CTE相关
  - id=76: 不同表的同名列的引用判定错误
  - id=77: 多删除了被外部JOIN ON使用的列
  - id=79: 同一个源表的两次使用, 对同名列的引用判断错误
  - id=82: 删除外层计算列后, 内层没有判断 计算引用的成分列 也可以删除
  - id=86: CTE相关
  - id=92: 不同表的同名列的引用判定错误
  - id=96: 多删除了 SELECT 1,1,1 的 1
  - id=97: 
    - 多删除了被外部JOIN ON使用的列
    - 不同表的同名列的引用判定错误
  - id=102: 最外层出现 "SELECT wen_zhang_zuixin.*", 需要保留子表的所有列
  - id=103: 不同表的同名列的引用判定错误
  - id=113: 
    - GT错误
    - 多删除了被外部JOIN ON使用的列
  - id=114: 不同表的同名列的引用判定错误
  - id=116: 多删除了被外部JOIN ON使用的列
  - id=119: CTE相关
  - id=136: 多删除了被外部 计算列 使用的列
  - id=137: 多删除了被外部 having 计算列 使用的列
  - id=146: 多删除了 SELECT 1,1,1 的 1
  - id=156: 多删除了被外部 计算列 使用的列
  - id=175: 多删除了被外部 GROUP BY 使用的列
  - id=182: 
    - GT错误, 没有删除`MAX(end_time)`
    - 多删除了被外部 WHERE (IS NOT NULL) 使用的列
  - id=183: GT错误
  - id=184: 多删除了被外部JOIN ON使用的列
  - id=187: 同一个源表的两次使用, 对同名列的引用判断错误
  - id=189: 同一个源表的两次使用, 对同名列的引用判断错误
  - id=209: 多删除了条件中的标量子查询中使用的嵌套变量
  - id=211: 
    - 多删除了被外部 计算列 使用的列
    - 多删除了条件中的标量子查询中使用的条件变量
  - id=224: 不同表的同名列的引用判定错误
  - id=234: 多删除了标量子查询的列
  - id=237: CTE相关
  - id=270: 漏删除同层条件列引用的变量
  - id=272: GT错误
  - id=273: 多删除了被外部 having 计算列 使用的列
  - id=274: 遗漏了条件列的级联删除
  - id=278: CTE相关
  - id=285: 不同表的同名列的引用判定错误
  - id=291: 多删除了 SELECT 1,1,1 的 1
  - id=297: 多删除了被外部 计算列 使用的列
  - id=300:
    - GT错误
    - 多删除了被外部ORDER BY使用的列

分类整理: 

```
# GT错误
id=14: GT对`eq.equipment_type`的判断错误
id=42: GT错误判断: `dx.shangpin_id` **不可删除**，因为它参与了 `GROUP BY` 
id=113: GT错误
id=182: GT错误, 没有删除`MAX(end_time)`
id=183: GT错误
id=272: GT错误
id=300: GT错误

# CTE相关
id=72: CTE相关
id=86: CTE相关
id=119: CTE相关
id=237: CTE相关
id=278: CTE相关

# SELECT 1
id=96: 多删除了 EXISTS(SELECT 1 ...)的1
id=146: 多删除了 EXISTS(SELECT 1 ...)的1
id=291: 多删除了 EXISTS(SELECT 1 ...)的1

# SELECT *
id=102: 最外层出现 "SELECT wen_zhang_zuixin.*", 需要保留子表的所有列

# 多删除了被外部 ? 使用的列
id=77: 多删除了被外部JOIN ON使用的列
id=97: 多删除了被外部JOIN ON使用的列
id=113: 多删除了被外部JOIN ON使用的列
id=116: 多删除了被外部JOIN ON使用的列
id=184: 多删除了被外部 JOIN ON使用的列
id=136: 多删除了被外部 计算列 使用的列
id=156: 多删除了被外部 计算列 使用的列
id=211: 多删除了被外部 计算列 使用的列
id=297: 多删除了被外部 计算列 使用的列
id=137: 多删除了被外部 having 计算列 使用的列
id=273: 多删除了被外部 having 计算列 使用的列

id=175: 多删除了被外部 GROUP BY 使用的列
id=182: 多删除了被外部 WHERE (IS NOT NULL) 使用的列
id=300: 多删除了被外部 ORDER BY使用的列

# 同名列的引用判定错误
id=76: 不同表的同名列的引用判定错误
id=92: 不同表的同名列的引用判定错误
id=97: 不同表的同名列的引用判定错误
id=103: 不同表的同名列的引用判定错误
id=114: 不同表的同名列的引用判定错误
id=224: 不同表的同名列的引用判定错误
id=285: 不同表的同名列的引用判定错误

id=79: 同一个源表的两次使用, 对同名列的引用判断错误
id=187: 同一个源表的两次使用, 对同名列的引用判断错误
id=189: 同一个源表的两次使用, 对同名列的引用判断错误

# 标量子查询
id=209: 多删除了条件中的标量子查询中使用的嵌套变量

# MISC
id=20: 窗口函数相关列, 不能正确判断
id=48: 对多表join的列的引用, 引用时没有使用表名, 识别错误
id=64: 对计算列和条件列引用判断错误, 少删除了
id=68: 与计算列有关的级联删除
id=82: 删除外层计算列后, 内层没有判断 计算引用的成分列 也可以删除
id=270: 漏删除同层条件列引用的变量
id=274: 遗漏了条件列的级联删除
id=211: 多删除了条件中的标量子查询中使用的条件变量
id=234: 多删除了标量子查询的列
``` 

构建补充数据, 使用提示词: 

```
我在训练一个大模型, 对SQL进行优化, 优化规则是: 
-----------
### 规则名称
删除无用的输出字段
### 规则描述
删除子查询中 没有被其他查询使用到的输出字段, 以减少查询的数据量。
只删除输出字段, 不要删除查询语句或造成其他影响。
-----------

在测试的过程中, 发现大模型在以下方面会优化错误, 我希望对模型进行一些针对性的训练, 帮我生成一个具有真实业务意义的针对这个优化错误的有挑战性的SQL:
- CTE表达式中使用的字段, 被外层查询的某一部分使用, 模型会错误地判断该字段没有使用, 导致错误地删除该字段
``` 

补充数据: 

  - "CTE表达式中使用的字段, 被外层查询的某一部分使用, 模型会错误地判断该字段没有使用, 导致错误地删除该字段": 3个
  - "大模型认为 `EXISTS(SELECT 1 ...)` 中的1 没有被外层引用, 所以多删除了 EXISTS(SELECT 1 ...)的1": 3个
  - "多删除了被外部JOIN ON使用的列": 3个
  - "多删除了被外部 计算列 使用的列": 3个
  - "多删除了被外部 having 计算列 使用的列": 2个
  - "不同表的同名列的引用判定错误, 进行了错误地删除": 3个
  - "同一个源表的两次引用, 比如 table_a as a和 table_a as c, 对两个引用使用的同名列的引用判断错误, 导致错删或者漏删了某列": 3个

# 训练21: 增加3个CTE数据和3个"`EXISTS(SELECT 1 ...)` "数据, 进行训练: 

训练过程: 

  - 第一阶段, GAE=8, OOM在epoch=28
  - 第二阶段, GAE=8, 从epoch=20续训, OOM在epoch=40
  - 第三阶段, GAE=1, 从epoch=30续训:
    - ![image2025-9-16 19:0:48.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-16%2019%3A0%3A48.png)
    - train/reward并不能完全收敛, eval/reward只能到0.8
    - 分析completion: 
      - 最大reward达不到1.0: prompt 1-4
        - prompt 1: 漏删列, 误认为被父查询JOIN引用 (machine_id)
        - prompt 2: 列被当前语句JOIN ON 引用, 错误识别成了被外层引用, 漏删
        - prompt 3: 多删了外层计算列引用的列
        - prompt 4: 评估数据: 
          - 级联删除 被弱化, 被 "虽然未被父级语句引用, 但其源列在当前语句的其他部分也没有被引用, 可以删除" 替代
      - reward长期达不到1.0: prompt 8/15/22/27
        - prompt 8: reward劣化: 列被当前语句JOIN ON 引用, 错误识别成了被外层引用, 漏删
        - prompt 15: reward劣化: 不同表的同名列, 误删
        - prompt 22: reward劣化: 漏删单独的一个列, 很奇怪, 没有特征
        - prompt 27: reward劣化, 突发一个低分, 不做深究
      - 全程0.95以上: prompt 10/17/21/23/29/32/34/35/37/38
    - 看上去 "级联删除"的逻辑被弱化, 而 "虽然未被父级语句引用, 但其源列在当前语句的其他部分也没有被引用, 可以删除" 的逻辑在加强

### 尝试一个更精简的训练策略

发现第一阶段训练, 到epoch=40时就会OOM

尝试调整 vllm_max_num_seqs=8, vllm_tensor_parallel_size=8, OOM在epoch=56

![image2025-9-17 0:44:12.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-17%200%3A44%3A12.png)

eval/reward仍有上升趋势, 但收敛比较慢

# 训练22-1: 只增强"级联删除"

使用训练20的32个数据 + 5个"级联删除"的数据, 进行训练

将vllm_max_num_seqs=8, vllm_tensor_parallel_size=8, 但将GAS=4 (否则会OOM)

参数: 

```
vllm_gpu_memory_utilization=0.8
 
第一阶段: 总144步, chord只配置了64步
--lr_scheduler_type "cosine_with_min_lr" \
--lr_scheduler_kwargs "'{\"min_lr\": 2e-5}'" \
--vllm_max_num_seqs 8 \
--vllm_tensor_parallel_size 8
--learning_rate "4e-5" \
--gradient_accumulation_steps 4 \
$(chord_args "$WORK_DIR/train_data.sft.jsonl" "64") \
--num_train_epochs "4" \

第二阶段: 总592步, chord配置了600步
--lr_scheduler_type "cosine_with_min_lr" \
--lr_scheduler_kwargs "'{\"min_lr\": 2e-5}'" \
--learning_rate "4e-5" \
--gradient_accumulation_steps 1 \
$(chord_args "$WORK_DIR/train_data.sft.jsonl" "600") \
--num_train_epochs "4" \
``` 

  - ![image2025-9-17 13:4:8.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-17%2013%3A4%3A8.png)
  - eval/reward 接近1.0, 但不稳定. 
  - 猜测: 在第一段可以使用GAE=4, 但在第二段需要拉长epoch
  - 训练时间: 6h+
  - 分析completion: 
    - 最大reward达不到1.0: prompt 1
      - prompt 1: GT错误: 

```
SELECT user_id, username FROM (SELECT user_id, username, email FROM (SELECT user_id, username, email, signup_date FROM Users WHERE signup_date > '2023-01-01') AS inner_query WHERE email LIKE '%@example.com') AS outer_query WHERE user_id IN (SELECT user_id FROM UserSubscriptions WHERE plan_id = (SELECT plan_id FROM SubscriptionPlans WHERE monthly_price * 12 < annual_price));
```

    - reward长期达不到1.0: prompt 2/3/5/23/24/26/28  

      - prompt 2: 末期可以达到大部分1.0. 其中低分的case的问题是: 漏删列, 误认为被父查询JOIN引用 (machine_id)
      - prompt 3: 多删除了 "列在外层JOIN...ON...中使用". 初期可以正确, 随着训练进行, 出现了多删除的问题. 符合训练预期 (并没有包括"列在外层JOIN...ON...中使用"的专项数据)
      - prompt 5: 评估数据: 末期的评分, 都是0.94或1.0. 低分的case, 主要是漏删了被当前层JOIN...ON引用的列 (v1.month)
      - prompt 23: reward下降: 展开了UNION父语句, 导致多一个删除, 不影响正确性
      - prompt 24: 漏删了被当前层where引用的列
      - prompt 26: 需要重新生成GT的结构: 

```
SELECT region, SUM(metric_value) AS total_value FROM (SELECT region, metric_value FROM (SELECT v.region, v.metric_value, r.country, c.description FROM visualization_data v JOIN region_info r ON v.region = r.region_name JOIN category_info c ON v.category = c.category_name WHERE v.data_date BETWEEN '2023-01-01' AND '2023-12-31') AS subquery1 WHERE country = 'USA') AS subquery2 GROUP BY region;
```

      - prompt 28: 漏删了被当前层where引用的列
    - 结论:
      - 增加"级联删除"的数据, 使用GAE=4, 是可以让模型收敛的
      - 评估中出现的问题, 大部分都是因为没有针对性的数据
      - 不认为训练有太大偏差, 可以继续增强数据

变更: 

  - 更新prompt 26的数据 (id=29)

# 训练22-2: 57条数据, 增强多个方向

![image2025-9-17 16:55:21.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-17%2016%3A55%3A21.png)

训练效果不好. 需要检查completion: 

  - reward完全为0: prompt 1/2
    - 两条都是不需要优化的数据, 但reward计算成0, 需要修正reward计算式
  - 最大reward达不到1.0: prompt 3
    - UNION父语句被展开, 导致多了两个删除
  - reward长期达不到1.0: prompt 6/7/9/13/25/26/30/31/33/36/48
    - prompt 6: 刚开始是满分, 训练后出现低分case, 删除了SELECT 1
    - prompt 7: 评估数据:
      - 训练后, 高分case, 还是漏删了被JOIN使用的条件列, 低分case, 不能进行级联删除
    - prompt 9: 刚开始是满分, 训练后出现低分case, 不能进行级联删除
    - prompt 13: UNION父语句被展开, 导致多了1个删除
    - prompt 25: UNION父语句被展开, 导致多了1个删除
    - prompt 26: 刚开始是满分, 训练后出现低分case, 不能进行级联删除
    - prompt 30: (id=29)的数据, 未修正
    - prompt 31: 满分case比例上升
    - prompt 33: 刚开始是满分, 训练后出现低分case, 不能进行级联删除
    - prompt 36: 刚开始是满分, 训练后出现低分case, 漏删列, 误认为被父查询JOIN引用 (machine_id)
    - prompt 48: GT错误, 存在重名列的语法错误: 

```
SELECT
    p.product_name,
    sp.salesperson_name,
    comp.current_month_amount,
    comp.previous_month_amount
FROM
    products AS p
JOIN
    (
        SELECT
            cur.product_id,
            cur.sales_amount   AS current_month_amount,
            cur.salesperson_id,
            cur.region_id,
            cur.units_sold,
            prev.sales_amount  AS previous_month_amount,
            prev.salesperson_id,
            prev.units_sold
        FROM
            monthly_sales AS cur
        JOIN
            monthly_sales AS prev ON cur.product_id = prev.product_id
        WHERE
            cur.sales_month = '2025-09' AND prev.sales_month = '2025-08'
    ) AS comp ON p.product_id = comp.product_id
JOIN
    salespersons AS sp ON sp.salesperson_id = comp.salesperson_id
WHERE
    comp.current_month_amount > comp.previous_month_amount * 1.5;
```

总结: 

  - 需要修复reward函数
  - 需要修复GT错误数据
  - 在训练中, 发现很多能力丢失 (当前评估的completion是第二段训练的completion, 其丢失了第一阶段的能力, 包括UNION父语句折叠/级联删除/JOIN...ON列/SELECT 1)
  - 很多case都是在第一阶段后满分, 随着第二阶段进行, 开始劣化

# 训练22-3: 57条数据, 增强多个方向, 第一阶段epoch=8

将第一阶段epoch从4调整成8, LR仍然是 4e-5 到 2e-5

蓝色曲线为epoch=4, 绿色曲线为epoch=8 (未完成). 虽然epoch增大, LR曲线下降变慢, 但在reward上没有突破.

![image2025-9-18 14:4:10.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-18%2014%3A4%3A10.png)

# 训练23: 在GRPO中测试vllm_quantization=int8

需要增加如下代码: 

![image2025-9-17 17:41:27.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-17%2017%3A41%3A27.png)

BTW: 因为A100原生不支持fp8, 所以会报错如下. 使用int8 (vllm_quantization='rtn'), 也会报错:

报错: 

```
File "/data/huangyan/ms-swift/swift/cli/rlhf.py", line 5, in <module>
 rlhf_main()
 File "/data/huangyan/ms-swift/swift/llm/train/rlhf.py", line 217, in rlhf_main
 return SwiftRLHF(args).main()
 File "/data/huangyan/ms-swift/swift/llm/base.py", line 49, in main
 result = self.run()
 File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 195, in run
 return self.train(trainer)
 File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 243, in train
 trainer.train(trainer.args.resume_from_checkpoint)
 File "/data/huangyan/ms-swift/swift/trainers/mixin.py", line 674, in train
 res = super().train(*args, **kwargs)
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2238, in train
 return inner_training_loop(
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2585, in _inner_training_loop
 tr_loss_step = self.training_step(model, inputs, num_items_in_batch)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 2202, in training_step
 return super().training_step(model, inputs, num_items_in_batch)
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 3805, in training_step
 inputs = self._prepare_inputs(inputs)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 169, in wrapper
 return func(self, *args, **kwargs)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 437, in _prepare_inputs
 generation_batch = self._generate_and_score_completions(generation_batch)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 169, in wrapper
 return func(self, *args, **kwargs)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 842, in _generate_and_score_completions
 inputs = self._generate_completions(inputs)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 823, in _generate_completions
 results = self._fast_infer(inputs)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 773, in _fast_infer
 self._move_model_to_vllm()
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 169, in wrapper
 return func(self, *args, **kwargs)
 File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 613, in _move_model_to_vllm
 llm_model.load_weights(state_dict.items())
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/model_executor/models/qwen3.py", line 321, in load_weights
 return loader.load_weights(weights)
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/model_executor/models/utils.py", line 291, in load_weights
 autoloaded_weights = set(self._load_module("", self.module, weights))
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/model_executor/models/utils.py", line 249, in _load_module
 yield from self._load_module(prefix,
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/model_executor/models/utils.py", line 222, in _load_module
 loaded_params = module_load_weights(weights)
 File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/model_executor/models/qwen2.py", line 403, in load_weights
 weight_loader = param.weight_loader
'Parameter' object has no attribute 'weight_loader'
``` 

尝试升级vllm到0.10.2

# 训练24: vllm_gpu_memory_utilization=0.5 与 0.8 对比

vllm_gpu_memory_utilization=0.8 的运行时长: 28,526s

vllm_gpu_memory_utilization=0.5 的运行时长: 27,967s

没有差异

前面是 vllm_gpu_memory_utilization=0.8的两阶段训练, 后面是 vllm_gpu_memory_utilization=0.5的两阶段训练:

0.5的eval/reward效果略差, 但不认为跟参数有关, 而是训练中的随机现象

![image2025-9-18 10:26:19.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-18%2010%3A26%3A19.png)

# 训练25: 57条数据, 增强多个方向

在训练22-2之上, 进行如下变更: 

  - 修复reward函数
  - 修复GT错误数据, 重建一个数据
  - 修复(id=29)的数据

在第二阶段开始时, 考虑将第一阶段评分低的case单独摘出来进行训练

RUNNING (ssh -p 24922 [root@183.196.130.56](<mailto:root@183.196.130.56>) )

# 训练26: CHORD衰减步数

在之前的训练中 (训练25), CHORD的衰减步数 = 总步数

分析CHORD论文:

是的，这篇论文提到了µ (mu) 值衰减所用的具体步数。

根据论文内容：

衰减步数：在第4.3节“Analysis on the Effects of µ”以及图7中明确提到，µ值是在前200个训练步骤中进行衰减的。具体来说，其值从0.9线性衰减到0.05，并在剩余的训练步骤中保持0.05不变。

总步数：在附录A.1“Hyperparameters”中提到，强化学习（RL）的最大训练步数设置为1,500步。

因此，µ值的衰减过程发生在前200步，占总训练步数（1,500步）的比例为 200/1500，约等于总步数的13.3%。

([183.196.130.56](<mailto:root@183.196.130.56>))

第一阶段: chord step = 384, 总 step = 224, GAS=4, 数据量=57

第二阶段: chord step = 600, 总 step = 912, GAS=1, 数据量=57

![image2025-9-19 13:41:8.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-19%2013%3A41%3A8.png)

chord覆盖所有step, 与下面的实验差异不大

(222.211.217.7)

第一阶段: chord step = 100, 总 step = 224, GAS=4, 数据量=57

第二阶段: chord step = 400, 总 step = 912, GAS=1, 数据量=57

![image2025-9-18 23:53:28.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-18%2023%3A53%3A28.png)

第一阶段已经过拟合

从step=70开始第二阶段续训, RUNNING(222.211.217.7):

![image2025-9-19 13:31:5.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-19%2013%3A31%3A5.png)

  - eval/reward 最高到0.92

(222.211.217.7)

对比试验: 将第一阶段epoch缩短至epoch=1

第一阶段: chord step = 100, 总 step = 56, LR从4e-5到2e-5, GAS=4, epoch=1, 数据量=57

第二阶段: chord step = 400, 总 step = 912, LR从4e-5到2e-5, GAS=1, epoch=4, 数据量=57

![image2025-9-19 13:34:23.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-19%2013%3A34%3A23.png)

  - eval/reward最高到0.9, 尚不稳定

对比试验: 将第一阶段epoch缩短至epoch=2 (与epoch=1放在一起进行对比), 没有明显优势

![image2025-9-19 23:33:16.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-19%2023%3A33%3A16.png)

对比试验: dora: eval收敛更快, 极值差不多

![image2025-9-20 11:4:22.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-20%2011%3A4%3A22.png)

# 训练27: vllm 使用 fp8 量化

无法达成, vllm没有动态量化功能

# 训练28: 对reward不达标的case进行针对性续训

基线: 训练26中 "对比试验: 将第一阶段epoch缩短至epoch=1"

![image2025-9-19 13:34:23.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-19%2013%3A34%3A23.png)

reward不达标的case ID: 43/13/3/34/22/35/11/56/44/27/49/10

安排续训: 

# 训练29: 使用GAS=2, epoch=6

第一阶段使用GAS=2 (epoch=6), 第二阶段使用GAS=1, 做两次实验:

  - 第一次 (绿色+白色曲线), LR从4e-5 -> 2e-5
  - 第二次 (蓝色+红色曲线), LR从4e-5 -> 0
    - 过拟合严重, eval/reward效果不好

将第一阶段epoch=6, 对eval没有提升

![image2025-9-20 11:14:45.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-20%2011%3A14%3A45.png)

# 训练30: 对于eval数据不达标的情况, 针对性训练

之前的训练中, completion中"级联删除"又被遗忘了, 只选取针对性数据进行训练: 

训练30-1: 

```
##########
# 训练数据, 只使用 "级联删除列" 和 "列在外层JOIN...ON...中使用"
# 使用 dora
# 第一阶段: GAE=4
# 第二阶段: GAE=1

``` 

![image2025-9-20 16:12:11.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-20%2016%3A12%3A11.png)

训练数据很快习得了 "级联删除"的特性, 但仍然会删除v1.month列 (外层使用了join...on...列)

修正提示词: 

```
修改前: 
  - "检查语句({语句名})的输出字段({字段内容}): 虽然未被父级语句引用, 但其源列在当前语句的({ON条件/WHERE条件等})中被引用: ({被引用的片段}), 且该输出字段是多余的, 可以删除"
  - "检查语句({语句名})的输出字段({字段内容}): 未被父级语句引用, 其源列在当前语句的其他部分也没有被引用, 可以删除"
 
修改后: 
  - "检查语句({语句名})的输出字段({字段内容}): 该字段未被父级语句引用。**但是**，其源列在当前语句的({WHERE条件/JOIN...ON条件/其他部分})中被引用({被引用的片段})用于({其作用})。由于该内部引用不要求其必须作为输出字段，因此可以删除
  - "检查语句({语句名})的输出字段({字段内容}): 该字段未被父级语句引用，**并且**其源列在当前语句的任何其他部分（如WHERE, JOIN, GROUP BY等）也**没有**被引用，可以删除"
``` 

TODO:

  1. 修正所有数据的提示词 
  2. 修正校验数据的提示词
  3. 重新生成所有数据的GT
  4. 检查所有标签"列在外层..."的数据正确性

使用修正后的数据进行训练: 

![image2025-9-21 9:56:27.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-21%209%3A56%3A27.png)

eval效果并不好, 分析completion:

  - 最大reward达不到1.0:
    - ID=22: 多删了外层条件列的引用列 (多个类似的列, 有一列判断错误)
    - eval:
      - 后期仍然是忘了"级联删除"
  - reward长期达不到1.0:
    - ID=12: GT中展开了UNION的父语句, 修正GT
    - ID=56: 后期是忘了"级联删除"

TODO:

  1. ID=24, 结构需要重新生成

# 训练31: 尝试只使用"级联删除"的训练数据

![image2025-9-21 14:18:9.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-21%2014%3A18%3A9.png)

对completion的分析, 训练数据都良好收敛, eval数据:

  - 在step=100时, 得分最高, 倾向于过度使用"级联删除"
  - 在step=256 (训练末期)时, 会有少使用"级联删除"的趋势

从 (第一次训练) step=100 (/data/huangyan/sql-ai-2/train_logs/2025-09-21_10-30-10/train_output/v1-20250921-111345/checkpoint-100)上, 增加续训, 使用"列在外层计算列使用"和"列在外层JOIN...ON...中使用"两部分数据

![image2025-9-21 20:1:26.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-21%2020%3A1%3A26.png)

分析completion: 

  - 在eval中, step=110时达到0.85, 其中并没有遗忘"级联删除", 但多删除"外部条件列使用" 和 "列在外层计算列使用"(但外部查询被删除) 的列

在(第二次训练) step=110 (/data/huangyan/sql-ai-2/train_logs/2025-09-21_18-38-02/train_output/v0-20250921-183816/checkpoint-110) 上增加续训, 使用所有训练数据

![image2025-9-21 21:40:40.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-21%2021%3A40%3A40.png)

  - 效果不好

在(第二次训练) step=110 (/data/huangyan/sql-ai-2/train_logs/2025-09-21_18-38-02/train_output/v0-20250921-183816/checkpoint-110) 上增加续训, 使用这几组数据: 

  - 列在外层JOIN...ON...中使用
  - 列在外层having使用
  - 列在外层计算列使用
  - 级联删除列

但效果不好: (白色曲线)

![image2025-9-21 23:40:56.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-21%2023%3A40%3A56.png)

检查completion: 

  - 其他规则(外层引用) 取代了部分的 "级联删除"
  - 感觉需要一个更复杂的涵盖多规则的case, 来教会模型区分各规则?

从 (第一次训练) step=100 (/data/huangyan/sql-ai-2/train_logs/2025-09-21_10-30-10/train_output/v1-20250921-111345/checkpoint-100)上, 增加续训, 

使用这几组数据: 

  - 列在外层JOIN...ON...中使用
  - 列在外层having使用
  - 列在外层计算列使用
  - 级联删除列

![image2025-9-22 7:6:13.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-22%207%3A6%3A13.png)

训练数据已经过拟合, 但评估效果不及在第一次训练上只使用 (列在外层JOIN...ON...中使用 + 级联删除列). 符合猜测: 模型不能处理多种优化之间的关系

直接使用这几组数据进行从头训练: 

  - 列在外层JOIN...ON...中使用
  - 列在外层having使用
  - 列在外层计算列使用
  - 级联删除列

![image2025-9-22 7:8:5.png](/assets/01KJBZZJ92R5GJ0VH1J4W3X93K/image2025-9-22%207%3A8%3A5.png)

从头训练, 效果不如分阶段训练. 在分阶段训练中, 第一阶段已经强调了 "级联删除列"这一优化. 从头训练中, 验证completion, 模型确实无法学习到"级联删除"这一技能.

# 总结

在整个过程中, 学习了如下调整: 

  1. 数据需要进行人为管理, 对于数据的变更需要人为复审, 防止训练数据腐烂. 少量的训练数据就可以达到很好的效果. 
  2. 刚开始使用的reward函数是多个reward函数, 用于控制模型的探索过程, 让其不要进入非法的探索步骤. 
  3. 但后来发现: 去掉sft阶段, 修改提示词, 让基模型从几个步骤模板中进行选择, 这样能让模型的探索更贴合基模型 (遗忘会更少)
  4. 于是简化reward函数, 只使用一个效果函数 (F1), 以及只使用grpo训练, 让模型自己进行探索和优化
  5. DAPO/CHORD等, 可以加速训练的收敛
  6. 因为一个偶然, 使用两阶段GRPO, 第一阶段使用GAS=4 (少量epoch), 第二阶段使用GAS=1, 可以完成一次reward高增长. 现在认为是GAS=4让模型总结了一个相对正确的方向, 然后用GAS=1探索细节
  7. 使用不同劣化方向的组合进行测试, 发现: 模型在现有数据(一个case针对一个劣化方向)下, 无法平衡/正确使用针对不同劣化方向的方法 (此起彼伏)
  8. TODO: 还是需要想办法生成对当前模型有挑战的case, 以当前的训练数据, 训练数据reward到高值时, eval数据并不能稳定居高
