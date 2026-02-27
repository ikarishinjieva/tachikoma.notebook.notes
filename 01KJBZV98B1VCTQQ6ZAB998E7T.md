---
title: 20250715 - 重新进行SQL优化的GRPO
confluence_page_id: 4161636
created_at: 2025-07-15T03:23:55+00:00
updated_at: 2025-07-27T12:39:11+00:00
---

# 当前发现的问题

之前的微调+GRPO, 是在JSON格式中进行的: 

  - 将SQL转成JSON格式, 在JSON中增加对重点区域的关注 (计算列, 对外引用列等)
  - 并在JSON结构中, 增加各层/子层的改写结果

使用JSON格式的主要原因是增强LLM对SQL层次的认知

之前的微调+GRPO, 可以对单次优化 (只使用一个规则优化一次) 达成良好效果.

想要在一次中 将一个规则进行多次优化, 发现效果不好.

主要原因是: 一个规则多次优化, 前面的优化会影响后面的优化, 所以需要对SQL结构进行多次回顾, 但JSON结构下, 只能进行一次回顾, 会导致效果漏掉优化点. 

# 下一步想法

  - 用JSON结构标注SQL结构 (增加结构编号), 增强LLM对结构的理解能力
  - 使用thinking, 完成对SQL结构的多次不同维度的回顾

考虑: 

  - 如何选择奖励函数

拟方案: 

  1. 在提示词中, 给出思考步骤的选择范围
     1. 对SQL(1.2)进行计算列分析
     2. 对SQL(1.2)进行列引用分析
     3. 对SQL(1.2)进行优化变更
     4. 对整个SQL从内向外进行顺序分析
     5. 反思变更是否正确
  2. 如何对思考步骤进行奖励
     1. 对GT进行变更列表, 比如: "删除 SQL (1.2) 的 xx 列"
     2. 在思考步骤中, 一旦做出了变更决定, 跟GT变更列表进行比对, 作为奖励

# 使用的机器

ssh -p 10931 [user01@Ncy2.ddnsto.net](<mailto:user01@Ncy2.ddnsto.net>) (8卡48G)

ssh -p 22206 [user01@125.71.97.102](<mailto:user01@125.71.97.102>) (4卡24G)

# 代码仓库

<https://github.com/ikarishinjieva/sql-ai-2>

# 构建训练数据

先使用以前构建的劣化的SQL数据: /opt/huangyan/sql-ai/v1/2_add_fault_in_sql.sqls.json

  - 先将SQL转成Json结构, 带标号
  - 然后使用制定思考步骤进行thinking扩展
  - 将思考步骤 + "反思要求", 让模型续写反思结果, 进行校验

挑选校验正确的数据, 作为训练数据.

# 尝试1 

sft: 20条数据

grpo: 另外32条数据

发现问题: 

  1. grpo阶段, 所有的reward都是1.0.
  2. 其中, 一半 都是因为对大模型输出的格式解析问题, 导致reward错误. 做了代码修订, 等待下一轮训练
  3. 另外一半, 确实是Qwen3-8B能正确的进行判断

# 尝试2

将训练数据进行过滤, 通过reward函数能解析的训练数据才能使用. 

sft: 10条数据

grpo: 另外18条数据

epoch = 4, num_generations = 2

![image2025-7-16 23:1:37.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-16%2023%3A1%3A37.png)![image2025-7-16 23:1:47.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-16%2023%3A1%3A47.png)![image2025-7-16 23:1:56.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-16%2023%3A1%3A56.png)

# 尝试3

在尝试2的基础上, 扩大epoch=16, num_generations = 8, 参数: 

```
    --num_generations 8 \
    --per_device_train_batch_size 4 \
    --num_train_epochs "16"
``` 

需要9h+ (24G 4090 * 4, 2个并发)

出现了OOM.

降低generation和batch, 仍然出现OOM:

```
    --num_generations 4 \
    --per_device_train_batch_size 2 \
    --num_train_epochs "16"
``` 

继续降低generation和batch, 相当于在尝试2基础上, 只增加epoch:

```
    --num_generations 2 \
    --per_device_train_batch_size 2 \
    --num_train_epochs "16"
``` 

  

仍然OOM

# 构建验证数据

构建过程中发现问题: 验证中出现了如下结果: 

```
5. 检查是否可以再次应用优化规则:
   - 检查子查询v1: 虽然它计算了bingoggr，但最终没有被使用，可以进一步优化
   - 检查子查询t1和v3: 它们的所有列都被使用(fd_ggr和fd用于计算fd_roi)
   - 可以再次应用优化规则: [删除(子查询v1)的列\"bingoggr\"]"}
``` 

但其并没有后续步骤, 导致对结果的判断有误

修正造数据的提示词, 要求: `在"检查是否可以再次应用优化规则"步骤中, 如果判断需要再次应用优化规则, 则需要续写后续步骤, 不要停在"检查是否可以再次应用优化规则"步骤`

出现了另外一组非法分析: 其中`应用优化规则：[删除子查询v2-1.1.3及其相关JOIN]`是一个模糊的概念. 

修正造数据时进行校验的匹配条件, 改为"对...应用优化规则"这种匹配

以上问题需要被修订

# 重新生成训练数据

  - 解决了"构建验证数据"中发现的问题
  - 生成数据时, 先进行随机, 否则数据相似度较高
  - 生成的数据, 需要reward function解析过滤一次, 确保其生成的内容可以被评分
  - 使用校验数据进行校验

# 尝试4

训练约5个小时左右, 跑完 7.65 个epoch, reward未见明显提升

![image2025-7-17 19:38:33.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-17%2019%3A38%3A33.png)![image2025-7-17 19:39:47.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-17%2019%3A39%3A47.png)

认为可能是LR较小, 将LR放大100倍到1e-4 (与sft相同)

# 尝试5

LR调整成1e-4, git (7c2233c24551f8b7e3ca048487b5c662c9dd8978)

steps=544, 耗时: 14h 19m 14s

![image2025-7-18 11:14:41.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-18%2011%3A14%3A41.png)

train/reward一直在上升, 曲线很漂亮, 结果也在变好.

eval验证数据评分没有提升, 查看实际效果: 

  - 提示词: 

```
 <|im_start|>user

我有一个SQL, 并对这个SQL进行了结构描述:

SQL:
-----------
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
) AS A44_T_1_    LIMIT 1 OFFSET 0
-----------

SQL的结构描述: 
-----------
{
  "SQL结构分析": {
    "主语句-1": {
      "SQL示意": "SELECT AVG(A44_T_1_.\"fd_roi\") AS \"T_A81_2_\" FROM (主FROM子语句-1.1) AS A44_T_1_ LIMIT 1 OFFSET 0",
      "业务含义": "计算平均ROI(投资回报率)指标，结果命名为T_A81_2_",
      "主要逻辑": [
        "从子查询A44_T_1_中获取fd_roi列",
        "计算这些ROI值的平均数",
        "限制只返回1条结果(第0条)"
      ],
      "子语句-1.1": {
        "主FROM子语句-1.1": {
          "SQL示意": "SELECT v1.months, v1.bingoggr, v1.valid_account, t1.fd_ggr, v3.fd, t1.fd_ggr/v3.fd as fd_roi, v2.cost FROM (子查询-1.1.1) v1 LEFT JOIN (子查询-1.1.2) t1 ON v1.months=t1.months LEFT JOIN (子查询-1.1.3) v2 ON v1.months=v2.state_date LEFT JOIN (子查询-1.1.4) v3 ON v1.months=v3.months",
          "业务含义": "构建包含月度指标、首存金额、ROI等关键指标的基础数据集",
          "主要逻辑": [
            "以v1(基础月度汇总数据)为主表",
            "左连接t1(当月首存用户产生的GGr)",
            "左连接v2(月度营销成本)",
            "左连接v3(月度首存用户数)",
            "计算ROI指标(fd_ggr/fd)",
            "返回包含所有关键指标的完整数据集"
          ],
          "子查询-1.1.1": {
            "SQL示意": "SELECT sum(bingoggr) as bingoggr, sum(valid_account) as valid_account, substring(pt,1,6) as months FROM dwd.dwd_order_detail_basic_ri GROUP BY substring(pt,1,6)",
            "业务含义": "计算每个月的总Bingo GGR和有效账户数",
            "主要逻辑": [
              "从订单明细表中聚合数据",
              "按月份(YYYYMM)分组",
              "计算每月的bingoggr总和",
              "计算每月的valid_account总和"
            ]
          },
          "子查询-1.1.2": {
            "SQL示意": "SELECT sum(bingoggr) as fd_ggr, substring(pt,1,6) as months FROM dwd.dwd_order_detail_basic_ri WHERE login_name IN (子查询-1.1.2.1) AND substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )=substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 ) GROUP BY substring(pt,1,6)",
            "业务含义": "计算当月首存用户产生的GGR(总赢额)",
            "主要逻辑": [
              "筛选当月首存的用户(通过子查询)",
              "筛选当月的交易记录",
              "按月份分组计算GGR总和"
            ],
            "子查询-1.1.2.1": {
              "SQL示意": "SELECT login_name FROM dwd.dwd_user_info_ri WHERE substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) =substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )",
              "业务含义": "获取当月首存的用户登录名",
              "主要逻辑": [
                "从用户信息表中选择用户",
                "筛选首存日期是当月的用户"
              ]
            }
          },
          "子查询-1.1.3": {
            "SQL示意": "SELECT sum(a.cost) as cost, a.state_date as state_date FROM (子查询-1.1.3.1) a GROUP BY a.state_date",
            "业务含义": "计算月度营销成本",
            "主要逻辑": [
              "合并按日统计和按月统计的成本数据",
              "按日期分组汇总总成本"
            ],
            "子查询-1.1.3.1": {
              "SQL示意": "SELECT sum(cost) as cost, state_date FROM dwd.dwd_market_channel_cost_di WHERE state_cycle='MONTH' GROUP BY state_date UNION ALL SELECT sum(cost) as cost, SUBSTRING(TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date FROM dwd.dwd_market_channel_cost_di WHERE state_cycle='DAY' GROUP BY SUBSTRING(TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)",
              "业务含义": "合并不同时间粒度的营销成本数据",
              "主要逻辑": [
                "第一部分：获取按月统计的成本",
                "第二部分：获取按日统计的成本并转换为月度格式",
                "使用UNION ALL合并两部分结果"
              ]
            }
          },
          "子查询-1.1.4": {
            "SQL示意": "SELECT substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months, count(1) as fd FROM dwd.dwd_user_info_ri GROUP BY substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )",
            "业务含义": "统计每月首存用户数量",
            "主要逻辑": [
              "从用户信息表中获取数据",
              "按首存月份分组",
              "计算每月的首存用户数"
            ]
          }
        }
      }
    },
    "总体业务含义": "这个SQL语句计算了平均ROI(投资回报率)指标，ROI的计算基于：当月首存用户产生的GGR除以当月首存用户数。数据来源包括：1)订单明细表中的GGR数据；2)用户信息表中的首存用户数据；3)营销渠道成本数据。该分析可能用于评估营销活动的投资回报效果，特别是在首存用户方面的表现。"
  }
}
-----------

现在我希望对这个SQL进行优化, 优化规则是: 
-----------
### 规则名称
投影下推

### 规则描述
删除子查询中无意义的列, 这些列在外查询中没有被使用。
-----------

现在我有一些可用的思考步骤: 
-----------
- 对整个SQL确定分析顺序: 先分析(子语句1.1), 然后分析(语句1), ...
- 对(子语句1.1)进行计算列分析: (子语句1.1)的列"CALC"是如何计算得来
- 对(子语句1.1)进行列引用分析: (子语句1.1)的列"COL2"被(语句1)引用
- 对(子语句1.1)应用优化规则: [删除(子语句1.1)的列"COL3", 删除(子语句1.1)的列"COL4"]
- 优化后的完整SQL
- 检查是否可以再次应用优化规则
-----------

输出格式如下: 
-----------
1. ...
	...

2. ...
	...

3. ...
	...
-----------

我现在的思考步骤如下: 
-----------

-----------

需要你从"可用的思考步骤"中的步骤, 续写思考步骤, 完成SQL的优化.
注意: 
1. 在"应用优化规则"步骤中, 需要严格遵守我规定的数组格式
2. 在"检查是否可以再次应用优化规则"步骤中, 如果判断需要再次应用优化规则, 则需要续写后续步骤, 不要停在"检查是否可以再次应用优化规则"步骤
3. 你需要严格遵循"优化规则"描述中的优化规则, 不要进行其他优化
<|im_end|>
<|im_start|>assistant

```

  - 效果: 

```
1. 对整个SQL确定分析顺序: 先分析(子语句1.1), 然后分析(语句1)
	- 优化顺序应从最内层的子查询开始，逐步向上分析，确保在上层查询中只保留必要的列

2. 对(子语句1.1)进行计算列分析: (子语句1.1)的列"fd_roi"是如何计算得来
	- fd_roi列在子查询中被定义为 t1.fd_ggr / v3.fd，这需要t1.fd_ggr和v3.fd列的值
	- 因此，fd_roi列的计算依赖于t1.fd_ggr和v3.fd列

3. 对(子语句1.1)进行列引用分析: (子语句1.1)的列"months"被(语句1)引用
	- 主查询中使用了子查询的months列来计算平均值
	- 主查询中的LIMIT 1 OFFSET 0实际上不会使用该列，但需要保留以确保正确计算

4. 对(子语句1.1)应用优化规则: [删除(子语句1.1)的列"bingoggr", 删除(子语句1.1)的列"valid_account", 删除(子语句1.1)的列"cost"]
	- 在子查询(子语句1.1)中，列bingoggr和valid_account仅在子查询内部使用，没有被主查询引用
	- 列cost仅在子查询(子语句1.1.3)中使用，没有被主查询引用
	- 因此可以删除这些列以减少数据传输和处理开销

5. 优化后的完整SQL
	- 优化后的SQL如下：
SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" FROM (
SELECT 
v1.months, 
t1.fd_ggr, 
v3.fd, 
t1.fd_ggr/v3.fd as fd_roi 
FROM 
(
SELECT 
sum(bingoggr) as bingoggr, 
sum(valid_account) as valid_account, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
GROUP BY 
substring(pt,1,6)
) v1 
LEFT JOIN 
(
SELECT 
sum(bingoggr) as fd_ggr, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
WHERE 
login_name IN (
SELECT 
login_name 
FROM 
dwd.dwd_user_info_ri 
WHERE 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) = 
substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
) 
AND substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )= 
substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
GROUP BY 
substring(pt,1,6)
) t1 
ON v1.months=t1.months 
LEFT JOIN 
(
SELECT 
sum(a.cost) as cost, 
a.state_date as state_date 
FROM 
(
SELECT 
sum(cost) as cost, 
state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='MONTH' 
GROUP BY 
state_date 
UNION ALL 
SELECT 
sum(cost) as cost, 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='DAY' 
GROUP BY 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
) a 
GROUP BY 
a.state_date
) v2 
ON v1.months=v2.state_date 
LEFT JOIN 
(
SELECT 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months, 
count(1) as fd 
FROM 
dwd.dwd_user_info_ri 
GROUP BY 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
) v3 
ON v1.months=v3.months
) AS A44_T_1_ LIMIT 1 OFFSET 0

6. 检查是否可以再次应用优化规则
	- 检查子查询中的列引用情况
	- 发现v1中的bingoggr和valid_account已被删除
	- v2中的cost列仍被使用，但主查询中没有引用该列
	- 可以继续优化v2子查询中的cost列
	- 优化规则可以再次应用

7. 对(子语句1.1.3)进行列引用分析: (子语句1.1.3)的列"cost"被(子语句1.1)引用
	- 主查询中的列"fd_roi"计算不依赖v2.cost列
	- 因此可以删除v2.cost列

8. 对(子语句1.1.3)应用优化规则: [删除(子语句1.1.3)的列"cost"]
	- 在子查询(子语句1.1.3)中，列cost在子查询内部使用，但主查询中没有引用该列
	- 因此可以删除该列以减少数据传输和处理开销

9. 优化后的完整SQL
	- 优化后的SQL如下：
SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" FROM (
SELECT 
v1.months, 
t1.fd_ggr, 
v3.fd, 
t1.fd_ggr/v3.fd as fd_roi 
FROM 
(
SELECT 
sum(bingoggr) as bingoggr, 
sum(valid_account) as valid_account, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
GROUP BY 
substring(pt,1,6)
) v1 
LEFT JOIN 
(
SELECT 
sum(bingoggr) as fd_ggr, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
WHERE 
login_name IN (
SELECT 
login_name 
FROM 
dwd.dwd_user_info_ri 
WHERE 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) = 
substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
) 
AND substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )= 
substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
GROUP BY 
substring(pt,1,6)
) t1 
ON v1.months=t1.months 
LEFT JOIN 
(
SELECT 
a.state_date 
FROM 
(
SELECT 
sum(cost) as cost, 
state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='MONTH' 
GROUP BY 
state_date 
UNION ALL 
SELECT 
sum(cost) as cost, 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='DAY' 
GROUP BY 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
) a 
GROUP BY 
a.state_date
) v2 
ON v1.months=v2.state_date 
LEFT JOIN 
(
SELECT 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months, 
count(1) as fd 
FROM 
dwd.dwd_user_info_ri 
GROUP BY 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
) v3 
ON v1.months=v3.months
) AS A44_T_1_ LIMIT 1 OFFSET 0

10. 检查是否可以再次应用优化规则
	- 检查子查询中的列引用情况
	- 发现v2中的state_date列仍被使用，主查询中没有引用该列
	- 但state_date列在v2中是被子查询(子语句1.1)引用的，用于连接
	- 因此不能删除state_date列
	- 检查v2中的其他列，发现没有其他列被引用
	- 因此无法继续应用优化规则

11. 优化完成
	- 当前SQL已经应用了投影下推规则，删除了无用列
	- 优化后的SQL更简洁，减少了数据传输和处理开销
	- 保持了原有的计算逻辑和结果准确性
	- 最终的SQL已满足优化要求
</think>

1. 对整个SQL确定分析顺序: 先分析(子语句1.1), 然后分析(语句1)
	- 优化顺序应从最内层的子查询开始，逐步向上分析，确保在上层查询中只保留必要的列

2. 对(子语句1.1)进行计算列分析: (子语句1.1)的列"fd_roi"是如何计算得来
	- fd_roi列在子查询中被定义为 t1.fd_ggr / v3.fd，这需要t1.fd_ggr和v3.fd列的值
	- 因此，fd_roi列的计算依赖于t1.fd_ggr和v3.fd列

3. 对(子语句1.1)进行列引用分析: (子语句1.1)的列"months"被(语句1)引用
	- 主查询中使用了子查询的months列来计算平均值
	- 主查询中的LIMIT 1 OFFSET 0实际上不会使用该列，但需要保留以确保正确计算

4. 对(子语句1.1)应用优化规则: [删除(子语句1.1)的列"bingoggr", 删除(子语句1.1)的列"valid_account", 删除(子语句1.1)的列"cost"]
	- 在子查询(子语句1.1)中，列bingoggr和valid_account仅在子查询内部使用，没有被主查询引用
	- 列cost仅在子查询(子语句1.1.3)中使用，没有被主查询引用
	- 因此可以删除这些列以减少数据传输和处理开销

5. 优化后的完整SQL
	- 优化后的SQL如下：
SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" FROM (
SELECT 
v1.months, 
t1.fd_ggr, 
v3.fd, 
t1.fd_ggr/v3.fd as fd_roi 
FROM 
(
SELECT 
sum(bingoggr) as bingoggr, 
sum(valid_account) as valid_account, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
GROUP BY 
substring(pt,1,6)
) v1 
LEFT JOIN 
(
SELECT 
sum(bingoggr) as fd_ggr, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
WHERE 
login_name IN (
SELECT 
login_name 
FROM 
dwd.dwd_user_info_ri 
WHERE 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) = 
substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
) 
AND substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )= 
substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
GROUP BY 
substring(pt,1,6)
) t1 
ON v1.months=t1.months 
LEFT JOIN 
(
SELECT 
sum(a.cost) as cost, 
a.state_date as state_date 
FROM 
(
SELECT 
sum(cost) as cost, 
state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='MONTH' 
GROUP BY 
state_date 
UNION ALL 
SELECT 
sum(cost) as cost, 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='DAY' 
GROUP BY 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
) a 
GROUP BY 
a.state_date
) v2 
ON v1.months=v2.state_date 
LEFT JOIN 
(
SELECT 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months, 
count(1) as fd 
FROM 
dwd.dwd_user_info_ri 
GROUP BY 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
) v3 
ON v1.months=v3.months
) AS A44_T_1_ LIMIT 1 OFFSET 0

6. 检查是否可以再次应用优化规则
	- 检查子查询中的列引用情况
	- 发现v1中的bingoggr和valid_account已被删除
	- v2中的cost列仍被使用，但主查询中没有引用该列
	- 可以继续优化v2子查询中的cost列
	- 优化规则可以再次应用

7. 对(子语句1.1.3)进行列引用分析: (子语句1.1.3)的列"cost"被(子语句1.1)引用
	- 主查询中的列"fd_roi"计算不依赖v2.cost列
	- 因此可以删除v2.cost列

8. 对(子语句1.1.3)应用优化规则: [删除(子语句1.1.3)的列"cost"]
	- 在子查询(子语句1.1.3)中，列cost在子查询内部使用，但主查询中没有引用该列
	- 因此可以删除该列以减少数据传输和处理开销

9. 优化后的完整SQL
	- 优化后的SQL如下：
SELECT AVG(A44_T_1_."fd_roi") AS "T_A81_2_" FROM (
SELECT 
v1.months, 
t1.fd_ggr, 
v3.fd, 
t1.fd_ggr/v3.fd as fd_roi 
FROM 
(
SELECT 
sum(bingoggr) as bingoggr, 
sum(valid_account) as valid_account, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
GROUP BY 
substring(pt,1,6)
) v1 
LEFT JOIN 
(
SELECT 
sum(bingoggr) as fd_ggr, 
substring(pt,1,6) as months 
FROM 
dwd.dwd_order_detail_basic_ri 
WHERE 
login_name IN (
SELECT 
login_name 
FROM 
dwd.dwd_user_info_ri 
WHERE 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) = 
substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
) 
AND substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )= 
substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
GROUP BY 
substring(pt,1,6)
) t1 
ON v1.months=t1.months 
LEFT JOIN 
(
SELECT 
a.state_date 
FROM 
(
SELECT 
sum(cost) as cost, 
state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='MONTH' 
GROUP BY 
state_date 
UNION ALL 
SELECT 
sum(cost) as cost, 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) as state_date 
FROM 
dwd.dwd_market_channel_cost_di 
WHERE 
state_cycle='DAY' 
GROUP BY 
SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
) a 
GROUP BY 
a.state_date
) v2 
ON v1.months=v2.state_date 
LEFT JOIN 
(
SELECT 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) as months, 
count(1) as fd 
FROM 
dwd.dwd_user_info_ri 
GROUP BY 
substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
) v3 
ON v1.months=v3.months
) AS A44_T_1_ LIMIT 1 OFFSET 0

10. 检查是否可以再次应用优化规则
	- 检查子查询中的列引用情况
	- 发现v2中的state_date列仍被使用，主查询中没有引用该列
	- 但state_date列在v2中是被子查询(子语句1.1)引用的，用于连接
	- 因此不能删除state_date列
	- 检查v2中的其他列，发现没有其他列被引用
	- 因此无法继续应用优化规则

11. 优化完成
	- 当前SQL已经应用了投影下推规则，删除了无用列
	- 优化后的SQL更简洁，减少了数据传输和处理开销
	- 保持了原有的计算逻辑和结果准确性
	- 最终的SQL已满足优化要求
```

发现的问题: 

  - 训练时, 处理有问题, 导致是在训练思考过程: 
    - 按照<https://qwen.readthedocs.io/en/latest/training/ms_swift.html> 的说明, 增加no_think
  - "对整个SQL确定分析顺序: 先分析(子语句1.1), 然后分析(语句1)"
    - 在SQL结构中, 会使用"子语句"和"子查询"等语句字样. 怀疑模型只能认知"子语句"的顺序
    - 确实看到了将"子查询-1.1.3", 描述成了"子语句-1.1.3"
    - 怀疑只能进行两层的语句顺序安排
    - 解决: 对提示词做了加强
  - 需要调整训练数据, 修复以上两个问题
  - 另外: "投影下推"会引起误解, 改成"删除无用的列查询"

# 尝试6: 重新生成训练数据, 并训练

选取了随机100个SQL进行生成, 总共生成了: 

  - 训练数据: sft 10条, grpo 38条
  - 验证集: 生成了最高质量的验证集: [analyzed_val_sql.json](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/analyzed_val_sql.json)

总共 step=304, 训练时间: 6h 41m 28s

![image2025-7-19 13:19:26.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-19%2013%3A19%3A26.png)

训练效果: 

  1. 虽然训练数据变少, 但与之前训练比起来, 校验reward有所提升 (验证集的数据质量变高, 更容易匹配)
  2. 训练和验证reward仍有上涨趋势

  

验证效果, 可能出现正确的情况, 但不稳定: 

num_generations=8, 只有1次1.0, 其他都是 -0.5 (都是格式问题, 提取出空操作)

```
DEBUG:reward_function:提取的操作 (模型): set()
DEBUG:reward_function:提取的操作 (基准): {'删除(子查询-1.1)的列"fd_ggr"', '删除(子查询-1.1)的列"months"', '删除(子查询-1.1)的列"valid_account"', '删除(子查询-1.1)的列"fd"', '删除(子查询-1.1)的列"cost"', '删除(子查询-1.1)的列"bingoggr"'}
DEBUG:reward_function:TP: 0, FP: 0, FN: 6
DEBUG:reward_function:计算分子: -3.0, 分母: 6
DEBUG:reward_function:裁剪前的分数: -0.5
DEBUG:reward_function:裁剪后的最终分数: -0.5
DEBUG:reward_function:全部分数计算完成: [-0.5, -0.5]
 
 
1. 对(子查询-1.1.1)进行列引用分析: (子查询-1.1.1)的列"months"被(子查询-1.1)引用作为连接条件, "bingoggr"和"valid_account"未被任何外层查询引用
	对(子查询-1.1.1)应用优化规则: [删除(子查询-1.1.1)的列"bingoggr", 删除(子查询-1.1.1)的列"valid_account"...
DEBUG:reward_function:提取的操作 (模型): {'删除(子查询-1.1)的列"fd_ggr"', '删除(子查询-1.1)的列"fd"', '删除(子查询-1.1)的列"months"', '删除(子查询-1.1)的列"bingoggr"', '删除(子查询-1.1)的列"valid_account"', '删除(子查询-1.1)的列"cost"'}
DEBUG:reward_function:提取的操作 (基准): {'删除(子查询-1.1)的列"fd_ggr"', '删除(子查询-1.1)的列"fd"', '删除(子查询-1.1)的列"months"', '删除(子查询-1.1)的列"bingoggr"', '删除(子查询-1.1)的列"valid_account"', '删除(子查询-1.1)的列"cost"'}
DEBUG:reward_function:TP: 6, FP: 0, FN: 0
DEBUG:reward_function:计算分子: 6.0, 分母: 6
DEBUG:reward_function:裁剪前的分数: 1.0
DEBUG:reward_function:裁剪后的最终分数: 1.0
DEBUG:reward_function:全部分数计算完成: [-0.5, 1.0]
``` 

# 修改训练数据生成时的验证方式

当前的方式: 使用一套提示词, 生成所有步骤, 然后用一个单独的步骤, 让模型续写"检查是否可以再次应用优化规则"的检查, 并判断检查结果

想做的修改: 

  1. 将三种检查都分别让模型进行续写
     1. 检查是否可以再次应用优化规则
     2. 检查所有的优化是否改变了SQL的结果
     3. 检查所有的优化是否存在超出优化规则的规定的情况
  2. 用另一个模型进行续写检查
  3. 在续写过程中, 要求给出明确的步骤结论. 如果检查失败, 则要求继续续写修订步骤, 直到成功, 或超过步骤限制
  4. 将续写过程中生成的结果, 作为训练数据
  5. 另外: 将analyze_sql.py中, 将数据筛选条件, 从 "改写两次" 改为 "改写至少一次"
  6. 使用随机200条SQL生成训练数据

  

总共193条数据, 将sft数据量增加到20条

# 尝试7: 使用新的训练数据

总共193条数据, 将sft数据量增加到20条, grpo数据为173条

训练时间: 21h 15m 56s, step: 1060/1376

commit: 0890f20ea3869702782376fd8aaf7398b2cacdf1

  

![image2025-7-20 11:14:41.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-20%2011%3A14%3A41.png)

  

观察到:

  - train/reward仍有上升趋势. 按运行时间太长, 需要减少数据量来快速试验
  - 校验中, 仍然是因为格式原因无法识别出有效信息

# 尝试8:

  - 训练数据量, 数据总量193个, sft用150个, grpo用 193个中的80个
  - ~~拆成两步课程训练, 先训练格式遵守, 再训练全数据~~
  - 放大sft的epoch=8, 让其更贴合格式
  - DAPO
  - 增加flash attention
    - 升级CUDA tookit 到 12.8 (使用runfile)
    - pip install flash-attn==2.8.0.post2 --no-build-isolation -i <https://pypi.tuna.tsinghua.edu.cn/simple>
  - ignore_data_skip, 需要在resume_only_model设置这个参数
  - 增加一些优化显存的参数

训练效果: 

训练总时间: 6h 30m 26s

train/reward有继续提升的趋势, 在eval上没有明显提升

![image2025-7-22 21:17:8.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-22%2021%3A17%3A8.png)

研究eval数据, 大量还是解析不出来答案

# 尝试9:

使用三步训练: 

  - sft
  - grpo: 训练格式保持
  - grpo: 训练SQL优化

但发现在第二步中, grpo大部分得到的评分是1.0, 评分初始就很高

高度测试sft后的模型, 在评估集上的性能: 

```
CUDA_VISIBLE_DEVICES=0 \
swift infer \
    --adapters /data/huangyan/sql-ai-2/train_logs/2025-07-22_21-23-29/train_output/v0-20250722-212508/checkpoint-144 \
    --infer_backend pt \
    --temperature 0 \
    --max_new_tokens 4096 \
    --val_dataset /data/huangyan/sql-ai-2/train_logs/2025-07-22_21-23-29/val_data.sft.jsonl \
    --max_batch_size 1
``` 

发现大部分情况下, 格式是正确的.

怀疑: 第三步grpo中的价值函数, 会导致模型牺牲 格式追求, 来换取得分

想到: 

  - 当格式错误时, 相当于没有提取任何有小动作, 评分大约是-0.5
  - 但如果格式正确, 提取的动作是错误的动作, 那么评分会是-1.0

修订这个价值函数设计, 增加对格式错误的惩罚, 进行重新训练

# 尝试10:

修改价值函数, 增加对格式错误的惩罚

训练效果: 

时长: 10h 52m 28s, step=470

![image2025-7-23 10:41:16.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-23%2010%3A41%3A16.png)

训练reward仍有提升趋势. 评估reward也有提升趋势.

对评估的结果进行分析: 

```
cat /data/huangyan/sql-ai-2/train_logs/2025-07-22_23-35-36/grpo_2025-07-22_23-42-45.log | grep 'reward_function:' | grep -B7 '收到 1 条待评估的结果'
``` 

评估中, 格式错误减少, 主要为出现在查询名匹配, 以及列名是否带表名前缀: 

```
INFO:reward_function:提取的操作 (模型): {'删除(子查询-1.1)的列"state_date"', '删除(子查询-1.1)的列"fd_ggr"', '删除(子查询-1.1)的列"months"', '删除(子查询-1.1)的列"bingoggr"', '删除(子查询-1.1)的列"cost"', '删除(子查询-1.1)的列"valid_account"', '删除(子查询-1.1)的列"fd"'}
INFO:reward_function:提取的操作 (基准): {'删除(子查询-1.1)的列"v1.bingoggr"', '删除(子查询-1.1)的列"v1.months"', '删除(子查询-1.1.1)的列"valid_account"', '删除(子查询-1.1)的列"v3.fd"', '删除(子查询-1.1)的列"t1.fd_ggr"', '删除(子查询-1.1)的列"v2.cost"'}
INFO:reward_function:TP: 0, FP: 7, FN: 6
INFO:reward_function:计算分子: -13.5, 分母: 13
INFO:reward_function:裁剪前的分数: -1.0384615384615385
INFO:reward_function:裁剪后的最终分数: -1.0384615384615385
INFO:reward_function:全部分数计算完成: [-1.0384615384615385]
INFO:reward_function:收到 1 条待评估的结果。
--
INFO:reward_function:提取的操作 (模型): {'删除(子查询-1.1.1)的列"bingoggr"', '删除(子查询-1.1.1)的列"valid_account"'}
INFO:reward_function:提取的操作 (基准): {'删除(子查询-1.1)的列"t1.fd_ggr"', '删除(子查询-1.1)的列"v1.months"', '删除(子查询-1.1)的列"v3.fd"', '删除(子查询-1.1)的列"v1.bingoggr"', '删除(子查询-1.1.1)的列"valid_account"', '删除(子查询-1.1)的列"v2.cost"'}
INFO:reward_function:TP: 1, FP: 1, FN: 5
INFO:reward_function:计算分子: -3.0, 分母: 7
INFO:reward_function:裁剪前的分数: -0.42857142857142855
INFO:reward_function:裁剪后的最终分数: -0.42857142857142855
INFO:reward_function:全部分数计算完成: [-0.42857142857142855]
INFO:reward_function:收到 1 条待评估的结果。
--
INFO:reward_function:提取的操作 (模型): set()
INFO:reward_function:提取的操作 (基准): {'删除(子查询-1.1)的列"v1.bingoggr"', '删除(子查询-1.1)的列"v1.months"', '删除(子查询-1.1)的列"t1.fd_ggr"', '删除(子查询-1.1.1)的列"valid_account"', '删除(子查询-1.1)的列"v3.fd"', '删除(子查询-1.1)的列"v2.cost"'}
INFO:reward_function:TP: 0, FP: 0, FN: 6
INFO:reward_function:计算分子: -15.0, 分母: 6
INFO:reward_function:裁剪前的分数: -2.5
INFO:reward_function:裁剪后的最终分数: -1.5
INFO:reward_function:全部分数计算完成: [-1.5]
INFO:reward_function:收到 1 条待评估的结果。
INFO:reward_function:提取的操作 (模型): {'删除(子查询-1.1.3)的列"state_date"', '删除(子查询-1.1.2)的列"fd_ggr"', '删除(子查询-1.1.4)的列"months"'}
INFO:reward_function:提取的操作 (基准): {'删除(子查询-1.1.1)的列"valid_account"', '删除(子查询-1.1)的列"v3.fd"', '删除(子查询-1.1)的列"v1.months"', '删除(子查询-1.1)的列"v2.cost"', '删除(子查询-1.1)的列"v1.bingoggr"', '删除(子查询-1.1)的列"t1.fd_ggr"'}
INFO:reward_function:TP: 0, FP: 3, FN: 6
INFO:reward_function:计算分子: -7.5, 分母: 9
INFO:reward_function:裁剪前的分数: -0.8333333333333334
INFO:reward_function:裁剪后的最终分数: -0.8333333333333334
INFO:reward_function:全部分数计算完成: [-0.8333333333333334]
INFO:reward_function:收到 1 条待评估的结果。
--
INFO:reward_function:提取的操作 (模型): {'删除(子查询-1.1)的列"v1.bingoggr"', '删除(子查询-1.1)的列"v2.cost"', '删除(子查询-1.1)的列"v1.valid_account"'}
INFO:reward_function:提取的操作 (基准): {'删除(子查询-1.1)的列"t1.fd_ggr"', '删除(子查询-1.1)的列"v3.fd"', '删除(子查询-1.1)的列"v1.bingoggr"', '删除(子查询-1.1.1)的列"valid_account"', '删除(子查询-1.1)的列"v1.months"', '删除(子查询-1.1)的列"v2.cost"'}
INFO:reward_function:TP: 2, FP: 1, FN: 4
INFO:reward_function:计算分子: -1.5, 分母: 7
INFO:reward_function:裁剪前的分数: -0.21428571428571427
INFO:reward_function:裁剪后的最终分数: -0.21428571428571427
INFO:reward_function:全部分数计算完成: [-0.21428571428571427]
INFO:reward_function:收到 1 条待评估的结果。
``` 

# 发现的问题

问题1: 校验数据中混入了联合操作 ('删除子查询v1的列"valid_account", 完全移除v2 LEFT JOIN')

```
DEBUG:reward_function:提取的操作 (模型): {'删除(子语句1.1)的列"months", "bingoggr", "valid_account", "fd_ggr", "fd", "cost", "state_date"'}
DEBUG:reward_function:提取的操作 (基准): {'删除子查询v1的列"bingoggr"', '删除子查询v1的列"valid_account", 完全移除v2 LEFT JOIN'}
``` 

  

问题2: 基准中出现了非法的语句编号: 

```
DEBUG:reward_function:提取的操作 (模型): {'删除(子语句1.1)的列"total_value"', '删除(子语句1.1)的列"max_time"', '删除(子语句1.1)的列"avg_value"'}
DEBUG:reward_function:提取的操作 (基准): {'删除(子查询-highlyRankedCells)的列"avg_value"', '删除(子查询-highlyRankedCells)的列"max_time"', '删除(子查询-highlyRankedCells)的列"total_value"'}
``` 

  

问题3: 如果语句中的列带有表名前缀, 那么基准中有可能有前缀, 有可能没有: 

```
DEBUG:reward_function:提取的操作 (模型): {'删除(子语句1.1)的列"instructor"', '删除(子语句1.1)的列"course_id"', '删除(子语句1.1)的列"term"', '删除(子语句1.1)的列"number"'}
DEBUG:reward_function:提取的操作 (基准): {'删除(子语句1.1)的列"course_offering.term"', '删除(子语句1.1)的列"subject_matter.course_id"', '删除(子语句1.1)的列"subject_matter.number"', '删除(子语句1.1)的列"course_offering.instructor"'}
``` 

  

类似的, 对于带别名的列, 两者输出也不统一

```
DEBUG:reward_function:提取的操作 (模型): {'删除(子语句1.1)的列"SYSDATE AS Query_Date"', '删除(子语句1.1)的列"ROWNUM AS Row_Number"'}
DEBUG:reward_function:提取的操作 (基准): {'删除(子语句1.1)的列"Row_Number"', '删除(子语句1.1)的列"Query_Date"'}

``` 

  

问题4: 语句匹配中, 会有引号差异: 

```
DEBUG:reward_function:提取的操作 (模型): {'删除(子语句1.1)的列company_records.address', '删除(子语句1.1)的列domain.name'}
DEBUG:reward_function:提取的操作 (基准): {'删除(子语句1.1)的列"company_records.address"', '删除(子语句1.1)的列"domain.name"'}
``` 

  

问题5: 动作不具体, 有概括性

```
DEBUG:reward_function:提取的操作 (基准): {'删除整个(CTE-1)定义，因为它未被引用'}
``` 

问题6: 错误的基准: 

```
DEBUG:reward_function:提取的操作 (基准): {'保持原样，所有列都被使用'}
``` 

问题7: 检查基准的查询命名: 

```
DEBUG:reward_function:提取的操作 (模型): {'删除(查询-1的CTE-1.1)的列"unused_column"', '删除(查询-1的CTE-1.1)的列"departure_time"', '删除(查询-1的CTE-1.1)的列"flight_id"'}
DEBUG:reward_function:提取的操作 (基准): {'删除(子查询-CTE-1.1)的列"flight.flight_id"', '删除(子查询-CTE-1.1)的列"departure_time"', '删除(子查询-CTE-1.1)的列"unused_column"'}
``` 

# 尝试11:

继续增强数据一致性: 

  - 修复 价值函数 的分数错误 (对于一些情况, 提取的动作数会少)
  - 生成数据时, 增强质量: 
    - 用qwen3-coder-plus-2025-07-22 从sql生成json结构
    - 用gemini-2.5-pro生成第一次的分析
    - 用deepseek-v3进行反思校验
    - 对"语句名称"和"列名", 进行更详细的限制
    - 对"动作"的定义, 进行更详细的限制
    - 在校验中, 对限制进行二次检查
  - 以上增强, 解决了上面"发现的问题"中的: 
    - 问题1/问题3/问题5/问题6, 通过校验来验证解决
    - 问题2/问题7, 修正了提示词, 但没有进行校验

训练commit: 2745c3941da06ac2e03b755bc798f416cd8050f4

训练效果: 

![image2025-7-26 1:48:32.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-26%201%3A48%3A32.png)

现象: 

  - train/reward 有上升趋势
  - eval/reward属于历史最好, 跟train/reward的趋势有点相似
  - grad_norm 普遍低于0.1, 中间有两次梯度突变, 分析见: [20250724 - 对GRPO训练中grad_norm的理解]

  

# 尝试12:

在尝试11的基础上, 继续训练. 

第一次使用LR=1e-4进行续训, eval/reward下降波动很厉害 (-0.1 下降到 -0.9), 于是停止训练:

![image2025-7-26 1:49:21.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-26%201%3A49%3A21.png)

  

更换LR=1e-5进行续训: 

train/reward和eval/reward都没有很好的进展: 

![image2025-7-26 12:35:23.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-26%2012%3A35%3A23.png)

  

更换LR=5e-5进行续训:

train/reward和eval/reward都没有很好的进展: 

![image2025-7-27 20:38:23.png](/assets/01KJBZV98B1VCTQQ6ZAB998E7T/image2025-7-27%2020%3A38%3A23.png)

  

考虑分析一下始终低分的训练数据, 看看当前的训练方向

# 下一轮待办

  - 发现一个现象: 删除列时, 会讲一些列删掉以后发现SQL不正确 (误删除了ORDER BY的列), 于是将ORDER BY也删掉. 下一轮需要做的: 
    - 对列引用分析, 进行重新定义: 对(子语句1.1)进行列引用分析: (子语句1.1)的列"COL2"被(语句1)引用, 或 (子语句1.1)的列"COL2"被当前语句的ORDER BY引用
    - 增加一个动作: 检查当前优化是否会破坏SQL的业务含义
  - ~~将analyze_sql.py中, 将数据筛选条件, 从 "改写两次" 改为 "改写至少一次"~~
  - ~~增加了生成数据时的验证过程, 验证训练数据是否能被reward function解析正确~~
  - ~~生成数据时, 先进行随机, 否则数据相似度较高~~
  - Qwen3训练时, 训练thinking过程?
  - ~~对于三个检查项:~~

```
- 检查是否可以再次应用优化规则 - 检查所有的优化是否改变了SQL的结果 - 检查所有的优化是否存在超出优化规则的规定的情况
```

    - ~~现在是做在一套提示词内. 需要拆成三个步骤, 让模型逐步添加~~
  - ~~DAPO~~
  - ~~课程训练, 先做格式训练, 再做整体训练~~
  - 使用DAPO的奖励函数: soft_overlong 限制模型输出长度
  - ~~ignore_data_skip, 需要在resume_only_model设置这个参数~~
  - 使用多次训练, 移动beta的中心, 是否有更好的效果
