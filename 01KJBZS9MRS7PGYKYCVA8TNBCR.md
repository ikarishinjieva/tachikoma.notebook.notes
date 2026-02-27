---
title: 20250328 - 研究SQL优化模型微调 - 处理错误case[2]
confluence_page_id: 3801317
created_at: 2025-03-28T05:23:11+00:00
updated_at: 2025-04-07T02:12:55+00:00
---

# 前继

使用不同的方法, 尝试使用规则"投影下推"优化SQL: 

```
SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" FROM (select
v1.months
,v1.bingoggr
,v1.valid_account
,t1.fd_ggr
,v3.fd
,t1.fd_ggr/v3.fd as fd_roi
,v2.cost
from
(
select
sum(bingoggr) as bingoggr,
sum(valid_account) as valid_account,
substring(pt,1,6) as months
from
dwd.dwd_order_detail_basic_ri
group by
substring(pt,1,6)
) v1
left join
(
select
sum(bingoggr) as fd_ggr,
substring(pt,1,6) as months
from
dwd.dwd_order_detail_basic_ri
where login_name in (
select login_name from dwd.dwd_user_info_ri where substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) =substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
)
and substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )=substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
group by
substring(pt,1,6)
) t1
on v1.months=t1.months
left join
(
 select sum(a.cost) as cost
,a.state_date as state_date
from
(
select sum(cost) as cost,state_date from dwd.dwd_market_channel_cost_di where state_cycle='MONTH' group by state_date
union all
select sum(cost) as cost, SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date from dwd.dwd_market_channel_cost_di where state_cycle='DAY' group by
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
) a
group by a.state_date
) v2
on v1.months=v2.state_date
left join
(
 SELECT
 substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months,
 count(1) as fd
FROM
 dwd.dwd_user_info_ri
GROUP BY
 substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
) v3
on v1.months=v3.months
) AS A44_T_1_ LIMIT 1 OFFSET 0

``` 

在之前 [20250325 - 研究SQL优化模型微调 - 处理错误case] 发现, 之前使用的"结构流程"是通过DS生成的, 其逻辑层次较乱, 准备使用gemini重新生成"结构流程"的训练数据. 

目前的训练思路: 

  - 让模型列出结构流程, 在结构流程里给出"改写指令"
  - 根据"改写指令" 改写 SQL

(之前是将 分析-改写指令-改写SQL 分为三步, 但三步之间的一致性无法保证. 于是改成两步, 并增加"结构流程"的逻辑性, 来试试是否可提高效果) 

# 使用gemini重新生成"结构流程"的训练数据, 进行微调

训练数据: 包括改写次数0/1/2的劣化SQL, 只使用规则"投影下推"的形式1(增加无用的列)进行劣化, 数据量: 1061

数据: /opt/huangyan/sql-ai/dataset/20250328/sql_optimize.jsonl.20250328

commit: 93d2c0ba882ddc3ca6f1983c6b261c9ae5ee27bf

| 训练 | 输出日志 | 分析 |
| --- | --- | --- |
| 模型: Qwen2.5-7B-Instruct-1Mepoch=4 | [1055.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/1055.log) |  |

  - 改写指令都得到了遵循
    - 其中删除了v2表的state_date, 会导致union all两侧的表列数不同, 导致报错
  - SQL中, 还多删除了v2.cost (v2是无用表, 但改写指令中没有指出)

  
模型: Qwen2.5-7B-Instruct-1Mepoch=8| [1106.txt](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/1106.txt) | 

  - 改写指令都得到了遵循
  - SQL中, 还多删除了第二层表的很多列(其中有删错的), 其在改写指令中不存在
  - 对v2表进行了连接键改写, 但改写的方式是错的

  
模型: Qwen2.5-7B-Instruct-1Mgradient_accumulation_steps=8epoch=4| [1118.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/1118.log) | 

  - 改写指令有一条没有被遵循, 因为其是错误的? :
    - t1的改写指令: 删除 `months` 列，因为它在外部查询中没有被使用。保留 `fd_ggr` 列用于计算ROI。
  - 改写指令中, 有一条分析错误: 删除了v2表的state_date, 会导致union all两侧的表列数不同, 导致报错
  - 改写的比较多, 目前效果比较好

  
模型: Qwen2.5-7B-Instruct-1Mgradient_accumulation_steps=8epoch=8| [1135.txt](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/1135.txt) | 

  - 改写指令只有一个, 得到了遵循
  - 分析深度不够

  
模型: Qwen2.5-7B-Instruct-1Mgradient_accumulation_steps=8epoch=16|  [1139.txt](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/1139.txt)| 

  - 改写指令都得到了遵循
    - 对v2表进行了连接键改写, 但改写的方式是错的
  - 结构分析中, 缺少了第二层的分析
  - SQL中, 还多删除了第二层表的很多列, 其在改写指令中不存在  

  
  
观察了比较常见的错误: 

  - 对于v2表进行了错误的改写 (因为v2表在外层的使用被删除, 所以整个表都不再被使用)
    - 删除了v2的连接键
  - 结构分析中, 缺少了第二层的分析 
    - 导致对v3表的fd列 是否被使用 产生了误判

先忽略其他错误. 尝试将训练数据中的结构分析, 定为从外到内, 广度搜索列举, 这样验证不会遗漏对第二层的分析, 再看是否会改善

# 重新构建数据, 让结构分析从外往内

commit: 059b57330581c2e053ae2ea4fc0b21e4ae9542ef

训练参数: model: Qwen/Qwen2.5-7B-Instruct-1M, epoch=4, gradient_accumulation_steps=8

训练效果: [1755.html](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/1755.html)

评估: 

  - 迭代1: 
    - 指令中要删除4列, 但`t1.fd_ggr`不应被删除, 实际也没有被删除
    - 指令中要求"删除此子查询中未在最外层查询中使用的列 `a.state_date`。", 但实际上删除了整个v2表
  - 迭代2:
    - 指令中要求: " 删除 `valid_account` 列，因为它没有在外部查询中使用。", 执行正确
    - 指令中错误分析: 集中于 用于group by的函数列
      - 改写指令: 删除 `substring(pt,1,6)` 列，因为它没有在外部查询中使用。

      - 改写指令: 删除 `TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD')` 列，因为它没有在外部查询中使用。

  - 迭代3: 
    - 指令中要求衍生表删除2列, 分析和执行正确
    - 指令中错误分析: 
      - 指令中要求"删除`t1.fd_ggr`列，因为它没有在外部查询中使用。", 分析是错的 (`t1.fd_ggr`是本轮删除的列之一, 但其还在函数列中使用, 判断错误). 执行时并没有删除
      - 与上一分析类似, "改写指令: 删除`v3.fd`列，因为它没有在外部查询中使用。" 也是错误的. 但确实被执行了
  - 迭代4:
    - 指令中要求v1删除1列, 分析和执行正确
    - 指令中错误分析:
      - 指令中要求删除t1.fd_ggr, 错误跟迭代3一样, 但实际被执行
  - 迭代5: 
    - 指令中错误分析:
      - 指令要求删除 子查询1和子查询2 的pt列, 类似于迭代2中的错误分析 (删除 `substring(pt,1,6)` 列)
      - 指令要求删除 子查询3 的first_deposit_date列, 跟上一错误类似, 都是group by中的substring的参数
    - 实际执行并没有改变SQL

总结: 

  - 对于v2表中的列处理问题, 需要补case: 在删除列后, 产生了无用表, 不应当删除表, 该怎么处理
  - 需要补充case: 对于 函数列中 "函数"涉及的列 的使用判断
  - 需要补充case: 对于 group by中使用了函数

### (增加epoch=6, 验证以上结论)

/opt/huangyan/sql-ai/output/Qwen2.5-7B-Instruct-1M/v20-20250328-182104/checkpoint-126

  

# 增强数据: 对于 函数列中 "函数"涉及的列 的使用判断  

增强数据1: 

  - 将SQL进行改造: "包括一个有子查询的SELECT(子查询有别名), 在SELECT中增加一个函数列, 函数列应当由 子查询中的多个列计算得来"
  - 对SQL进行劣化
  - 对劣化后的SQL, 生成"结构流程"和"优化流程", 在生成时提出额外要求: "你需要格外强调: 函数列使用到了其他哪些表的哪些列"

  

进行微调: Qwen2.5-7B-Instruct-1M, 原数据+增强数据1, epoch=4, gradient_accumulation_steps=8

推理效果: [0011.html](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/0011.html)

效果说明: 并没有体现出对计算字段的说明或特殊操作

迭代1:  
1\. 共删除了四列, 其中三列正确  
2\. 多删除了一列state_date (group by的列), 在分析中并没有  
3\. 其中并没有指出计算字段的特殊性

迭代2:  
1\. 删除了valid_acount, 分析和执行 正确  
2\. 删除了v2表, 在分析中并没有  
3\. 其中并没有指出计算字段的特殊性

迭代3:  
1\. 删除了v3列, 分析错误, 其是计算字段的一部分  
2\. 分析中要求删除fd_ggr, 分析错误, 但并没有执行  
3\. 分析中要求删除bingoggr, 分析正确, 但并没有执行  
4\. 其中并没有指出计算字段的特殊性

迭代4:  
1\. 分析中要求删除子查询的fd_ggr, 分析错误, 没有被执行  
2\. 其中并没有指出计算字段的特殊性

进行微调: Qwen2.5-7B-Instruct-1M, 原数据+增强数据1, epoch=6, gradient_accumulation_steps=8

推理效果: [0021.html](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/0021.html)

效果说明:并没有体现出对计算字段的说明或特殊操作

迭代1:  
1\. 共删除了四列, 其中三列分析和执行正确  
2\. 对于衍生表的valid_account, 并没有分析, 但执行中删除正确  
3\. 其中并没有指出计算字段的特殊性

迭代2:  
1\. 分析中要求删除fd_roi, 分析错误, 实际没有执行  
2\. 其中并没有指出计算字段的特殊性

看上去增强的数据并没有作用, 需要检查数据, 发现增强数据的数量错误, 只有5个. 扩大到50个, 重新进行实验. 

  

commit: b52aa5a693c7d032bee1b140dadc1c443b4069a3

推理效果: 

  - Qwen2.5-7B-Instruct-1M, 原数据+增强数据1, epoch=4, gradient_accumulation_steps=8  

    - [6_evaluate.1.1653.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/6_evaluate.1.1653.log)  

  - Qwen2.5-7B-Instruct-1M, 原数据+增强数据1, epoch=6, gradient_accumulation_steps=8
    - [6_evaluate.2.1653.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/6_evaluate.2.1653.log)

结论: 增强数据中, 都包括明确的对函数列的分析, 但在推理结果中, 未见对函数列的分析

  

下一步: 

  - 看起来并没有学习到特征, 尝试将gradient_accumulation_steps降下来, 进行测试
    - gradient_accumulation_steps=1时, 函数列的分析, 偶尔会出现在输出的SQL中. 需要检查增强数据  

  - 将"计算列"的分析作为独立的章节, 让其在增强数据中更明确
  - 重新生成所有训练数据, 如有函数列, 都增加对函数列的分析

  

将"计算列"的分析作为独立的章节, 让其在增强数据中更明确.

重新训练, 在推理结果中仍然看不到"计算列". 确认增强数据中存在计算列. 原数据和增强数据的数量比为 350:50, 只能认为原数据影响过大, 需要验证这个猜想.

  - 只用增强数据进行训练, 发现推理结果中仍然没有"计算列". 检查增强数据, 其中输出带有"计算列"有36个 (总数据量50个). 
  - 将这36个数据过滤出来, 单独作为一个训练集训练. 微调后推理结果中仍然没有"计算列", 而包括了章节号等特征. 
    - 尝试增加epoch=8, 学到了"让我们像剥洋葱一样" 这个特征, 仍然没有"计算列"
    - 尝试增加epoch=12, 仍然没有"计算列"
  - 在推理提示词中, 增加提示'注意关注"计算列"', 推理结果中出现"计算列"
    - 判断: 是因为"计算列"在长上下文中不受关注, 在数据量比较少的情况下, 学习不到"计算列"这个特征.
    - 但当使用全数据集时, 即使增加了对于"计算列"的提示, 推理中仍然不能出现
  - 还是考虑洗所有数据

发现evaluate代码有问题, evaluate用的模型和训练用的模型不同.

已经洗了所有数据, 继续进行评估: 

模型=Qwen2.5-7B-Instruct-1M

| 训练参数 | 训练loss | 包含"计算列"的个数 |
| --- | --- | --- |
| gradient_accumulation_steps=1, epoch=4 | 0.07540373 | 19 |
| gradient_accumulation_steps=1, epoch=8 | 0.00921984 | 20 |
| gradient_accumulation_steps=8, epoch=4 | 0.25078831 | 1 |
| gradient_accumulation_steps=8, epoch=8 | 0.14352549 | 3 |
| gradient_accumulation_steps=8, epoch=16 | 0.01015371 | 31 |
  
结论: gradient_accumulation_steps=8 需要更多的epoch才能收敛. 其是否找到了"更正确"的方向, 需要之后评估.

评估以上训练的效果: 

  - gradient_accumulation_steps=1, epoch=4
    - 评估日志: [2025-03-31_11-48-58.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-03-31_11-48-58.log)
    - 迭代1:
      - 正确执行: 删除 v1.bingoggr, v1.valid_account, v2.cost 列，因为它们没有在外部查询中使用。  
正确分析未执行: 删除 bingoggr 和 valid_account 列，因为它们没有在外部查询中使用。   
涉及v2表, 分析不完全未执行: 删除 state_date 列，因为它没有在外部查询中使用。  
涉及v2表, 分析不完全未执行: 删除 inner query 中的 state_date 列，因为它在外部查询中没有被使用。

      - 未分析但错误修改: 将v3.fd修改成了t1.fd
    - 迭代2:
      - 分析模糊, 未执行: 删除 v1、t1、v2 和 v3 中未在外部查询或连接条件中使用的列。  
分析正确且执行正确: 删除 `valid_account` 列  
涉及v2表, 分析错误, 但执行: 删除 `a.state_date` 列  
涉及v2表, 分析错误, 未执行: 删除第二个 UNION ALL 分支中的 `state_date` 列

    - 迭代3: 
      - 改写指令: 删除 v1, t1, v2, v3 中未被使用的列  
改写指令: 删除 v1 中未被使用的列，即保留 bingoggr 和 months 列  
改写指令: 删除 t1 中未被使用的列，即保留 fd_ggr, fd, fd_roi 列  
改写指令: 删除 t1.1 中未被使用的列，即保留 fd_ggr 和 months 列  
改写指令: 删除 t1.2 中未被使用的列，即保留 cost 列  
改写指令: 删除 t1.2.1 中未被使用的列，即保留 cost 和 state_date 列  
改写指令: 删除 t1.3 中未被使用的列，即保留 fd 列  
改写指令: 删除 t1.3.1 中未被使用的列，即保留 months 和 fd 列  
改写指令: 删除 v2 中未被使用的列，即保留 cost 列  
改写指令: 删除 v2.1 中未被使用的列，即保留 cost 和 state_date 列  
改写指令: 删除 v3 中未被使用的列，即保留 months 和 fd 列

      - 分析了很多, 但分析仅描述了现状, 并没有明确指令
  - gradient_accumulation_steps=1, epoch=8
    - 评估日志: [2025-03-31_12-17-18.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-03-31_12-17-18.log)
    - 迭代1: 
      - 分析正确, 但只修改了前两个: 删除 v1.bingoggr, v1.valid_account, v2.cost 列，因为它们没有在外部查询中使用。  
分析模糊: 保留 fd_roi 计算列.  
分析正确, 执行正确: 删除 valid_account 列.  
分析模糊: 保留 cost 计算列.  
分析模糊: 保留 months 和 fd 计算列.

    - 迭代2:

      - 分析正确, 执行正确: 删除 `t1.fd_ggr` 和 `v2.cost` 列，因为它们没有在外层查询中使用。
    - 迭代3:
      - 涉及v2表, 分析但未执行: 删除 v2 衍生表中的 `cost` 列，因为它没有被外层查询使用.  
涉及v2表, 分析错误, 已执行: 删除子查询中的 `state_date` 列, 因为 v2 衍生表中使用了 `a.state_date`, 所以需要保留子查询中的 `a.state_date` 列.  
涉及v2表, 分析错误, 已执行: 第二个 SELECT 语句中的 `state_date` 列, 变为 `NULL` 因为 v2 衍生表中使用了 `a.state_date`, 所以需要在子查询中使用 `NULL` 代替 `state_date`.

    - 迭代4:
      - 分析正确, 执行正确: 删除 `bingoggr` 列  
分析正确(但列名有歧义), 执行正确: 删除 `bingoggr` 列  
涉及v2表, 分析正确未执行: 删除 `a.cost` 列  
涉及v2表, 分析正确未执行: 删除 `a.cost` 列

    - 迭代5:
      - 所有分析均没有修改, 执行也没有修改
  - gradient_accumulation_steps=8, epoch=16
    - 评估日志: [2025-03-31_13-35-24.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-03-31_13-35-24.log)
    - 迭代1: 
      - 分析正确, 执行正确: 删除 v1.bingoggr, v1.valid_account, v2.cost 列，因为它们没有在外层查询中使用。 保留 t1.fd_ggr, v3.fd 用于计算 fd_roi。  
分析正确, 执行正确: 删除 bingoggr 和 valid_account 列，因为它们没有在外部查询或连接条件中使用。 保留 months 用于分组和连接。  
分析错误, 未执行, 涉及计算列(但分析中已经明确标记了计算列): 删除 fd_ggr 列，因为它在后续连接中没有被使用。 保留 months 用于分组和连接。  
涉及v2表, 分析错误, 未执行: 删除 state_date 列，因为它在外部查询或连接条件中没有使用。 保留 cost 用于计算 ROI。

      - 将v3.fd错误改写成了t1.fd

    - 迭代2: 
      - 分析正确, 执行时多了一个逗号: 删除 t1.fd, v1.months 两列，因为它们没有在 fd_roi 计算中使用，也没有在外部查询中使用。保留 v1.months 是为了保持连接的完整性。  
分析正确(前一迭代改错了列名, 此处将错就错): 删除 t1.fd 列，因为它没有在外部查询中使用。  
分析正确, 执行正确: 删除 v2.cost 列，因为它没有在外部查询中使用。只保留 v2.state_date 列。  
分析错误, 未执行: 删除 v3.months 列，因为它没有在外部查询中使用。只保留 v3.fd 列。

    - 迭代3: 
      - 分析错误, 未执行: 删除 v3.1 生成的 fd 列，因为它未被外层查询使用.  
分析错误, 未执行: 删除 months 列，因为它未被外层查询使用.

  

结论: 

  - 对于gradient_accumulation_steps=1, epoch=4/8, 其结果中: 
    - 不再出现 跟计算列相关的错误 (错误地删除了计算列涉及的列)
    - 分析错误都涉及了v2表
    - 存在 "分析但不执行" 的情况
    - 存在"将v3.fd修改成了t1.fd"这种错误
  - 对于gradient_accumulation_steps=8, epoch=16, 结果会更差:   

    - 出现了 跟计算列相关的错误 (错误地删除了计算列涉及的列)  

    - 出现了SQL格式错误
    - ...
  - 增强数据 (实际上是重新洗了数据, 在数据中增加了"计算列"的描述) 是有用的
  - 认为在这种小数据量 (300-400)下, gradient_accumulation_steps=8的效果比gradient_accumulation_steps=1更差, 需要更长的时间进行收敛

  

下一步: 通过数据增强, 正确处理v2表

  

# 通过数据增强, 正确处理v2表

v2表不一定能被安全删除. 

所以需要处理的是: 让模型不要去修改join on...的字段, 而不是让模型删除v2表

  

增加增强数据 (让模型不要去删除join on...的字段)

  - commit : 5f90a61b66644a2c73485a49148f7b9fdd7baec0
  - 增强脚本: /opt/huangyan/sql-ai/enhance_data_1.ipynb
    - 注意: 需要增加多个验证, 确保增强出的数据中, 明确有"连接列"的分析, 并且明确在推理过程中包含了具体列的分析信息
  - 增强数据量: 38条
  - 训练参数: gradient_accumulation_steps=1, epoch=8
    - epoch=4时, 没有效果
  - 需要在评估脚本中, 增加"4\. 在结构分析中, 你需要关注连接列" 的要求
    - 如果不增加要求, 则在epoch=8时无法展现效果
    - 可以认为是跟增强数据量少有关, 导致增强进去的知识不明显, 需要明确的提示

增强效果: 模型能意识到连接条件不能被删除, 从而保留"state_date"字段

````
......
5. **内层查询 v2 (SELECT ... FROM ... GROUP BY ...): 计算每月成本**

   SQL示意:
   ```sql
   SELECT sum(a.cost) as cost
   FROM (
       ...
   ) a
   GROUP BY a.state_date
   ```

   5.1. **计算列 `cost`**:  这一列是通过`sum(a.cost)`计算得出的，表示每个月的总成本。

   改写分析: `cost`列被外层查询使用，`state_date`列作为连接条件使用，都不能删除。

   改写指令: 无需改写
......
```` 

结论: 

  - 增强数据可以解决连接列被误删除的问题

TODO:

  - 在生成完整数据集时, 加入 对连接列的分析要求
  - 增大增强数据的数量, 尝试 不增加提示 就可以改变模型的思考流程
  - 继续分析输出 /opt/huangyan/sql-ai/eval_logs/2025-04-01_17-05-25.log , 找到下一个要解决的问题

  

# 找到下一个要解决的问题

评估日志: [2025-04-01_17-05-25.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-01_17-05-25.log)

分析: 

  - 迭代1: 
    - 改写分析: 外层查询只使用了`fd_roi`列，`v1.bingoggr`、`v1.valid_account`、`t1.fd_ggr`、`v2.cost`没有被使用。因此，子查询中除了这些列以及`fd`、`months`列之外的列都可以删除。

      - 分析正确, 避开了计算列, 并且要求删除的列比较全
    - 但实际分析未执行, SQL并没有修改
  - 迭代2:
    - 改写分析: `v1.bingoggr`, `v1.valid_account`, `t1.fd_ggr`, `v2.cost` 在外层查询中没有被使用，可以删除。
      - 分析正确
      - 但实际执行时, 还多删除了两列 (正确)
      - 错误: 将"t1.fd_ggr/v3.fd AS fd_roi" 改成了 "t1.fd_roi", 这一步没在分析中出现
        - 其在对输入部分进行SQL示意时, 就已经丢失了计算列, 跟后续分析无关
    - 改写分析: `state_date` 在外层查询中没有被使用，可以删除。
      - 分析错误, 分析中没有提及"连接列", 导致此处有误 (但在迭代1中提及了"连接列"且分析正确, 认为已具备能力, 但稳定性需要提高)
  - 迭代3:
    - 改写指令: 删除`valid_account`列，保留`bingoggr`和`months`列。

      - 分析正确, 执行正确
  - 迭代4: 
    - 分析没有需要修改, 执行无误
    - 分析中也出现了关于"连接列"的描述 (正确)

TODO: 

  - 修改推理要求, 要求改写指令不能使用模糊代词, 比如: "改写指令: 根据子查询的改写结果，删除不必要的列。"
  - 将"连接列"的训练数据增多, 使其稳定
  - 让分析和修改统一, 使用DPO? 
    - 先分析训练数据, 查看其是否 分析和修改 统一
    - 训练数据中有多行SQL, 但之改写其中最后一行, 需要清洗

  

# DPO尝试

commit: 7ece1ba84058834f470fe04695c0118b09068e10

生成DPO数据脚本: /opt/huangyan/sql-ai/dpo_data.ipynb

思路: 将SQL优化流程中, "改写指令"中的一部分删除, 构成劣化数据. 用于DPO训练. 

训练期待效果: 

  - 在基准测试中, 有 "不在改写指令中提及的" SQL改变
  - 在DPO后的模型中, 不存在

  

训练前的评估结果: [2025-04-02_18-05-22.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-02_18-05-22.log) (只关注 "不在改写指令中提及的" SQL改变)

  - 迭代1: 
    - (不确定) * **删除`v1.bingoggr`在SELECT子句中的引用**: 虽然分析过程2.2 指令保留了`v1.bingoggr`，但最终生成的SQL中，在最外层SELECT子句中删除了对它的引用。这与分析过程的描述略有不符。分析过程强调保留参与衍生表构建和JOIN操作的列，但没有明确说明是否需要在最终SELECT子句中保留。尽管如此，由于`v1.bingoggr`没有被最终结果使用，删除它在SELECT子句中的引用是合理的优化，并不算错误。
    - (正确) * **其他改写**: Diff中的其他改写，例如保留`v1.valid_account`、`v2.cost`等，都与分析过程的指令一致。

  - 迭代2: 
    - (正确) * Diff中删除了`A44_T_1_`中`v1.valid_account`列，这在1.1的改写指令中明确给出。
    - (正确) * Diff中删除了`v1`中`SUM(valid_account) AS valid_account`，这在1.1.1的改写指令中明确给出。

    - (正确) * Diff中没有改动`t1`，`v2`，`v3`，这与1.1.2，1.1.3，1.1.4的“无需改写”指令一致。

  - 迭代3: 
    - (错误) * **衍生表 A44_T_1_:** Diff 中删除了 `t1.fd_ggr`, `v3.fd` 和 `fd_roi` 的计算，分析过程只给出了删除 `t1.fd_ggr` 和 `v3.fd` 的指令，删除 `fd_roi` 的计算没有对应的指令。

    - (错误, 但不属于 "不在改写指令中提及的" SQL改变) * **衍生表 v1:** Diff 中没有改动，而分析过程给出了删除 `bingoggr` 的指令，改写没有执行。

    - (错误, 但不属于 "不在改写指令中提及的" SQL改变) * **衍生表 t1:** Diff 中没有改动，而分析过程给出了删除 `fd_ggr` 的指令，改写没有执行。

    - (正确) * **衍生表 v2 和 v3:** Diff 中没有改动，分析过程的指令也是无需改写，两者一致。

  - 迭代4:
    - (不确定) * **v1 中 `substring(pt, 1, 6)` 的变化:** Diff 将 `v1.months` 的来源从一个独立的列变成了直接使用 `substring(pt, 1, 6)`。分析过程中虽然提到了 `months` 列，但没有明确指出要直接用 `substring(pt, 1, 6)` 替换 `v1.months` 的选择方式。**没有明确给出**

    - (正确) * **其他改写:** 除了上述 v1 的变化以及指令 4 的错误外，其余 diff 中的改动都与分析过程中的指令对应。

  - 迭代5:
    - “对SQL改写的diff” 为 None，表示没有改写。 “对改写的分析过程”中所有步骤的分析结论均为“无需改写”，因此，diff 中的空改写与分析过程中的指令一致。

DPO, epoch=4

训练后的评估结果: [2025-04-02_18-22-55.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-02_18-22-55.log)

  - 迭代1:
    - (正确) * **删除 `v1.bingoggr`, `v1.valid_account`, 和 `v2.cost`**: 这三个列的删除在 1.1 衍生表 A44_T_1_ 的分析中有明确的指令。

    - (不重要的错误) * **调整 `SELECT` 子句中列的顺序**: diff 中 `SELECT t1.fd_ggr, v1.months, v3.fd, t1.fd_ggr/v3.fd AS fd_roi` 的顺序与原SQL不同。分析过程中没有提到这个改动，也没有给出相应的指令。

  - 迭代2:
    - (不确定) * **diff1:** 衍生表 A44_T_1_ 的 SELECT 子句从 `t1.fd_ggr, v1.months, v3.fd, t1.fd_ggr/v3.fd AS fd_roi` 改为 `t1.fd_ggr/v3.fd AS fd_roi`。  
* **分析过程:** 对应指令1，但指令1要求删除 `t1.fd_ggr` 和 `v3.fd`，而 diff 中保留了它们用于计算 `fd_roi`。虽然结果正确，但描述不够精确。**部分正确**

    - (不确定) * **diff2:** v2 子查询的 SELECT 子句从 `sum(a.cost) AS cost, a.state_date AS state_date` 改为 `sum(a.cost) AS cost`，并去除了最外层的 `GROUP BY a.state_date`。  
* **分析过程:** 对应指令4，但指令4只提到了删除 `cost` 和 `state_date` 列，没有明确说明去除了 `GROUP BY`。**部分正确**

  - 迭代3:
    - (错误) * **Diff 1:** 将最外层嵌套查询中的 `v1` 改名为 `t1`。**没有明确给出。** 虽然分析过程中提到了 t1，但并没有明确指示要将 v1 改名为 t1。
  - 迭代4:
    - (正确) 由于 Diff 为 None，表示没有进行任何改写。而“对改写的分析过程”中，除了最外层查询的指令是“无需改写此部分”外，衍生表部分的指令是需要删除部分列，这与 Diff 中无改动的情况相矛盾。

从评估结果上看出: 

  - 训练后的错误, 有一处明确的"删除了列但指令中没提及"
  - 训练后的错误/不确定的项, 都不是"删除了列但指令中没提及"

再有更多测试之前, 不确定DPO是否真的起了作用.

hint: DPO可能内存不够, 使用deepspeed zero3可减少内存, 但需要去除 --gradient_checkpointing_kwargs '{\"use_reentrant\": false}' \\. 否则会报错:

[rank1]: torch.utils.checkpoint.CheckpointError: torch.utils.checkpoint: Recomputed values for the following tensors have different metadata than during the forward pass.

查看最长列排序

awk '{ print length, $0 }' "/opt/huangyan/sql-ai/dataset/dpo.jsonl" | sort -nr | head -n 10 | cut -d " " -f1

去除最长列

awk -v n="15000" 'length <= n' "/opt/huangyan/sql-ai/dataset/dpo.jsonl" > "/opt/huangyan/sql-ai/dataset/dpo.truncate.jsonl"

从make_dataset的数据集中, 抽前200条数据, 进行增强, 增强方向: 

  - 在SQL中增加指令中不存在的变化
  - 在指令中增加SQL中不存在的变化
  - 数据总量: 332

结果分析: 

  - sft后: 
    - [2025-04-05_12-41-16.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-05_12-41-16.log)
  - data: 1-200, epoch=8
  - DPO后:   

    - [2025-04-05_13-23-42.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-05_13-23-42.log)

未见明显提升. 或者说在DPO前的记录就没有太大问题 (一两处指令和SQL不一致).

尝试修改提示词, 增加要求: "5. 一定一定一定要让"改写指令"和"改写后的SQL"完全匹配", 未见改观

增加明确性的要求: 

```
1. 对SQL的结构进行分析, 并给出改写指令
    - 改写指令一定要明确, 举例: 
        - 明确的指令(明确了操作类型, 对象名称, 规格等): 添加列abc, 添加索引def
        - 不明确的指令: 添加符合条件的列, 并添加响应索引
``` 

仍然没有改善.

对DPO数据进行评估, 使用评估脚本: /opt/huangyan/sql-ai/evaluate_dpo_data.ipynb

对前20条数据进行评估, 发现劣化数据质量并不好: [evaluate_dpo_data.2124.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/evaluate_dpo_data.2124.log)

调整生成脚本, 对生成数据进行评估, 抛弃评估不通过的数据

commit: 1343417d66d611b36982b2c27d7731def89b6124

重新组织DPO数据后, 使用了100个数据生成DPO, 总数据量194, 增强方向:

  - 在SQL中增加指令中不存在的变化
  - 在指令中增加SQL中不存在的变化
  - 确认: DPO前评估是一致的, DPO后评估是不一致的

评估效果: 

  - LR=1e-4, epoch=8
    - [2025-04-06_02-46-10.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-06_02-46-10.log)
    - ![image2025-4-6 13:11:12.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2013%3A11%3A12.png)![image2025-4-6 13:11:22.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2013%3A11%3A22.png)![image2025-4-6 13:11:27.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2013%3A11%3A27.png)
  - LR=1e-5, epoch=8
    - [2025-04-06_03-18-43.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-06_03-18-43.log)
    - ![image2025-4-6 13:12:12.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2013%3A12%3A12.png)![image2025-4-6 13:12:17.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2013%3A12%3A17.png)![image2025-4-6 13:12:20.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2013%3A12%3A20.png)

尝试将sft和DPO的数据分开

  - 在生成数据后, 用100-n数据来进行sft
  - 然后用0-100数据来进行DPO
  - 扩大梯度累积gradient_accumulation_steps=16, LR=1e-4, epoch=32
  - 训练效果: 
    - ![image2025-4-6 21:14:48.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2021%3A14%3A48.png)![image2025-4-6 21:14:52.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2021%3A14%3A52.png)![image2025-4-6 21:14:55.png](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/image2025-4-6%2021%3A14%3A55.png)
    - margin能升高到20, 但无法再扩大了
  - 评估效果: [2025-04-06_19-57-02.log](/assets/01KJBZS9MRS7PGYKYCVA8TNBCR/2025-04-06_19-57-02.log)
    - 结果已经偏差很大了, 一开始就会将SQL进行大幅缩减, 也就是说DPO学得了其他倾向

调整DPO的参数: beta和temperature, 可以影响曲线, 但最终效果无法达成.

尝试输入 SQL+改写过程, 不输入优化规则, 也不强调优化只强调改写, 让Qwen2原模型进行改写, 发现仍然无法完整遵循

输入: 

````
我有一个SQL和对SQL的改写过程: 
 
SQL
----------
SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" FROM (select
v1.months
,v1.bingoggr
,v1.valid_account
,t1.fd_ggr
,v3.fd
,t1.fd_ggr/v3.fd as fd_roi
,v2.cost
from
(
select
sum(bingoggr) as bingoggr,
sum(valid_account) as valid_account,
substring(pt,1,6) as months
from
dwd.dwd_order_detail_basic_ri
group by
substring(pt,1,6)
) v1
left join
(
select
sum(bingoggr) as fd_ggr,
substring(pt,1,6) as months
from
dwd.dwd_order_detail_basic_ri
where login_name in (
select login_name from dwd.dwd_user_info_ri where substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) =substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
)
and substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )=substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
group by
substring(pt,1,6)
) t1
on v1.months=t1.months
left join
(
 select sum(a.cost) as cost
,a.state_date as state_date
from
(
select sum(cost) as cost,state_date from dwd.dwd_market_channel_cost_di where state_cycle='MONTH' group by state_date
union all
select sum(cost) as cost, SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date from dwd.dwd_market_channel_cost_di where state_cycle='DAY' group by
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
) a
group by a.state_date
) v2
on v1.months=v2.state_date
left join
(
 SELECT
 substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months,
 count(1) as fd
FROM
 dwd.dwd_user_info_ri
GROUP BY
 substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
) v3
on v1.months=v3.months
) AS A44_T_1_ LIMIT 1 OFFSET 0
----------
 

改写过程: 
----------
1. **最外层查询 (SELECT AVG(...) ... FROM ...): 计算FD ROI的平均值**

   SQL示意:
   ```sql
   SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" 
   FROM (
       ...
   ) AS A44_T_1_
   LIMIT 1 OFFSET 0;
   ```

   计算列 `T_A81_2_`: 这一列是由`AVG(A44_T_1_."fd_roi")`计算得出的，表示所有月份的FD ROI的平均值。

   改写分析:  最外层查询只使用了`fd_roi`列，因此衍生表中其他未使用的列可以删除。

   改写指令:  无

1.1. **衍生表 (FROM (... LEFT JOIN ...)): 准备所需数据**

   SQL示意:
   ```sql
   FROM (
       SELECT v1.months, v1.bingoggr, v1.valid_account, t1.fd_ggr, v3.fd, t1.fd_ggr/v3.fd as fd_roi, v2.cost
       FROM (
           ...
       ) v1
       LEFT JOIN (
           ...
       ) t1 ON v1.months = t1.months
       LEFT JOIN (
           ...
       ) v2 ON v1.months = v2.state_date
       LEFT JOIN (
           ...
       ) v3 ON v1.months = v3.months
   ) AS A44_T_1_
   ```

   计算列 `fd_roi`: 这一列是由`t1.fd_ggr/v3.fd`计算得出的，表示FD的ROI（Return on Deposit）。

   改写分析: 衍生表中`bingoggr`, `valid_account`, `fd_ggr`, `cost`列在外层查询中没有被使用。

   改写指令: 删除`v1.bingoggr`, `v1.valid_account`, `t1.fd_ggr`, `v2.cost` 列。

1.1.1. **第一个内层查询 (SELECT ... FROM ... GROUP BY ...): 统计每月的 Bingo GG 和有效账户数量**

   SQL示意:
   ```sql
   SELECT SUM(bingoggr) AS bingoggr, SUM(valid_account) AS valid_account, SUBSTRING(pt, 1, 6) AS months
   FROM dwd.dwd_order_detail_basic_ri
   GROUP BY SUBSTRING(pt, 1, 6)
   ```

   计算列 `months`: 这一列是由`SUBSTRING(pt, 1, 6)`计算得出的，表示年月信息。

   改写分析:  `bingoggr` 和 `valid_account` 列在外层查询中没有被使用。

   改写指令: 删除`v1.bingoggr`, `v1.valid_account` 列。

1.1.2. **第二个内层查询 (SELECT ... FROM ... WHERE ... AND ... GROUP BY ...): 计算每月 FD GGR**

   SQL示意:
   ```sql
   SELECT SUM(bingoggr) AS fd_ggr, SUBSTRING(pt, 1, 6) AS months
   FROM dwd.dwd_order_detail_basic_ri
   WHERE login_name IN (...) AND SUBSTRING(TO_CHAR(current_date::timestamp, 'YYYYMMDD'), 1, 6) = SUBSTRING(TO_CHAR(pt::timestamp, 'YYYYMMDD'), 1, 6)
   GROUP BY SUBSTRING(pt, 1, 6)
   ```

   计算列 `months`: 这一列是由`SUBSTRING(pt, 1, 6)`计算得出的，表示年月信息。

   改写分析: `fd_ggr` 列在外层查询中没有被使用。

   改写指令: 删除`t1.fd_ggr` 列。

1.1.3. **第三个内层查询 (SELECT ... FROM (... UNION ALL ...): 合并每日和每月成本数据**

   SQL示意:
   ```sql
   SELECT SUM(a.cost) AS cost, a.state_date AS state_date
   FROM (
       ...
   ) a
   GROUP BY a.state_date
   ```

   计算列 `state_date`: 这一列在第一个子查询中由`SUBSTRING(TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)`计算得出，在第二个子查询中直接使用`SUBSTRING(TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)`。在衍生表中重命名为`state_date`。

   改写分析: `cost` 列在外层查询中没有被使用。

   改写指令: 删除`v2.cost` 列。

1.1.4. **第四个内层查询 (SELECT ... FROM ... GROUP BY ...): 统计每月 FD 数量**

   SQL示意:
   ```sql
   SELECT SUBSTRING(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'), 1, 6) AS months, COUNT(1) AS fd
   FROM dwd.dwd_user_info_ri
   GROUP BY SUBSTRING(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'), 1, 6)
   ```

   计算列 `months`: 这一列是由`SUBSTRING(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'), 1, 6)`计算得出的，表示年月信息。
   计算列 `fd`: 这一列是由`COUNT(1)`计算得出的，表示每月 FD 的数量。

   改写分析: `fd` 列在外层查询中没有被使用。

   改写指令: 删除`v3.fd` 列。

----------

需要你严格遵守改写过程, 对SQL进行改写, 并输出改写后的完整的SQL.
注意: 
1. 只输出SQL, 不要输出其他内容
2. 严格遵守改写指令, 不要遗漏, 也不要做出超出改写指令的变更
```` 

原模型输出: 

````
```sql
SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" 
FROM (
    SELECT v1.months, t1.fd_ggr/v3.fd as fd_roi
    FROM (
        SELECT SUM(bingoggr) as bingoggr,
               SUM(valid_account) as valid_account,
               SUBSTRING(pt,1,6) as months
        FROM dwd.dwd_order_detail_basic_ri
        GROUP BY SUBSTRING(pt,1,6)
    ) v1
    LEFT JOIN (
        SELECT SUM(bingoggr) as fd_ggr,
               SUBSTRING(pt,1,6) as months
        FROM dwd.dwd_order_detail_basic_ri
        WHERE login_name IN (
            SELECT login_name FROM dwd.dwd_user_info_ri 
            WHERE substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) = 
                  substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
        )
        AND substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 ) = 
               substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
        GROUP BY SUBSTRING(pt,1,6)
    ) t1 ON v1.months = t1.months
    LEFT JOIN (
        SELECT SUM(a.cost) as cost,
               a.state_date as state_date
        FROM (
            SELECT SUM(cost) as cost, state_date 
            FROM dwd.dwd_market_channel_cost_di 
            WHERE state_cycle='MONTH' 
            GROUP BY state_date
            UNION ALL
            SELECT SUM(cost) as cost, SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date 
            FROM dwd.dwd_market_channel_cost_di 
            WHERE state_cycle='DAY' 
            GROUP BY SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
        ) a
        GROUP BY a.state_date
    ) v2 ON v1.months = v2.state_date
    LEFT JOIN (
        SELECT substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months,
               count(1) as fd
        FROM dwd.dwd_user_info_ri
        GROUP BY substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
    ) v3 ON v1.months = v3.months
) AS A44_T_1_
LIMIT 1 OFFSET 0;
```
```` 

将现在的目标修正为: SQL+改写指令, 让模型进行遵循, 从sft从头开始构建数据

# 收尾

将本次使用的脚本, 收纳在v2目录中
