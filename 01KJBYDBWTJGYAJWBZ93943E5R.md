---
title: 20210531 - Clickhouse Join 研究
confluence_page_id: 1146896
created_at: 2021-05-31T05:48:03+00:00
updated_at: 2021-06-13T15:50:22+00:00
---

# TODO

  - <https://www.jianshu.com/p/363d734bdc03>
    - 当两表关联查询只需要从左表出结果时，建议用IN而不是JOIN，即写成SELECT ... FROM left_table WHERE join_key IN (SELECT ... FROM right_table)的风格。

      - (实验: IN与JOIN的不同)
    - 不管是LEFT、RIGHT还是INNER JOIN操作，小表都必须放在右侧。因为CK默认在大多数情况下都用hash join算法，左表固定为probe table，右表固定为build table且被广播。

      - <https://fuxkdb.com/2020/08/28/2020-08-28-ClickHouse%E6%9F%A5%E8%AF%A2%E5%88%86%E5%B8%83%E5%BC%8F%E8%A1%A8LEFT-JOIN%E6%94%B9RIGHT-JOIN%E7%9A%84%E5%A4%A7%E5%9D%91/>
    - CK的查询优化器比较弱，JOIN操作的谓词不会下推，因此一定要先做完过滤、聚合等操作，再在结果集上做JOIN。这点与我们写其他平台SQL语句的习惯很不同，初期尤其需要注意。

      - (实验: JOIN谓词不下推)

    - 两张分布式表上的IN和JOIN之前必须加上GLOBAL关键字。如果不加GLOBAL关键字的话，每个节点都会单独发起一次对右表的查询，而右表又是分布式表，就导致右表一共会被查询N2次（N是该分布式表的shard数量），这就是所谓的查询放大，会带来不小的overhead。加上GLOBAL关键字之后，右表只会在接收查询请求的那个节点查询一次，并将其分发到其他节点上。

  - <https://tech.youzan.com/clickhouse-zai-you-zan-de-shi-jian-zhi-lu/>
    - ClickHouse 分布式 Join 处理方式不进行 Shuffle exchange， 不适合数据量大的情况。
    - 有些 SQL 语法，比如当 Join 的左表是 subquery，而不是表的时候，ClickHouse 无法进行分布式 Join，只能在分布式表的 Initiator 的单节点进行 Join。  
详情请见: <https://github.com/ClickHouse/ClickHouse/issues/9477>
  - JOIN table engine
  - External dictionaries
  - join_algorithm的各种值
    - partial_merge
    - auto
    - prefer_partial_merge
  - join相关的各项配置

# 知识

  - <https://clickhouse.tech/docs/en/sql-reference/statements/select/join/>
    - ASOF JOIN: 适用于数字/时序等 不完全相等 的列的JOIN
      - 举例: 

```
SELECT expressions_list
FROM table_1
ASOF LEFT JOIN table_2
ON equi_cond AND closest_match_cond
``` 
      - ASOF JOIN 会根据 closest_match_cond (table1.a <= table2.b), 找到最匹配的记录 (最接近判断条件的记录), 进行JOIN
    - Join始终在WHERE和聚合之前
      - TODO: 观察?
    - 内存限制参数
      - max_rows_in_join
      - max_bytes_in_join
      - join_overflow_mode: THROW or BREAK
      - TODO: 待测试行为
        - (After some threshold of memory consumption, ClickHouse falls back to merge join algorithm) ??
    - 对NULLable列的处理参数: join_use_nulls
      - TODO: 待测试
  - <https://cloud.tencent.com/developer/article/1621346>
    - TODO: 使用GLOBAL JOIN 和 不使用GLOBAL的JOIN, 流程区别
    - TODO: GLOBAL JOIN 使用的内存临时表的配置??
    - TODO: 查询放大 的 现象

# 实验条件

21.6.3.14

造表:

```
//a表
localhost :) show create table a;

SHOW CREATE TABLE a

Query id: 636d95bc-15dd-4423-8775-7c1a8f03e584

┌─statement──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ CREATE TABLE default.a
(
    `a` Int32,
    `b` Int32,
    `c` Int32
)
ENGINE = MergeTree
ORDER BY a
SETTINGS index_granularity = 8192 │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

1 rows in set. Elapsed: 0.007 sec.

//b表
localhost :) show create table b;

SHOW CREATE TABLE b

Query id: a2dd9ff1-8356-42bc-b45f-2765a365c937

┌─statement──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ CREATE TABLE default.b
(
    `a` Int32,
    `b` Int32,
    `d` Int32
)
ENGINE = MergeTree
ORDER BY b
SETTINGS index_granularity = 8192 │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

1 rows in set. Elapsed: 0.009 sec.
``` 

# 实验: 读懂JOIN的执行计划

```
localhost :) explain actions=1 select * from a join b on a.b = b.b where d = 1;

EXPLAIN actions = 1
SELECT *
FROM a
INNER JOIN b ON a.b = b.b
WHERE d = 1

Query id: e9f20ee6-e306-41b3-ba5f-011d20bd4d49

┌─explain────────────────────────────────────────────────────────────────────────────────────────┐
│ Expression ((Projection + Before ORDER BY))                                                    │
│ Actions: INPUT :: 0 -> a Int32 : 0                                                             │
│          INPUT :: 1 -> b Int32 : 1                                                             │
│          INPUT :: 2 -> c Int32 : 2                                                             │
│          INPUT :: 3 -> b.a Int32 : 3                                                           │
│          INPUT :: 4 -> b.b Int32 : 4                                                           │
│          INPUT :: 5 -> d Int32 : 5                                                             │
│ Positions: 0 1 2 3 4 5                                                                         │
│   Filter (WHERE)                                                                               │
│   Filter column: equals(d, 1) (removed)                                                        │
│   Actions: INPUT :: 0 -> a Int32 : 0                                                           │
│            INPUT :: 1 -> b Int32 : 1                                                           │
│            INPUT :: 2 -> c Int32 : 2                                                           │
│            INPUT :: 3 -> b.a Int32 : 3                                                         │
│            INPUT :: 4 -> b.b Int32 : 4                                                         │
│            INPUT : 5 -> d Int32 : 5                                                            │
│            COLUMN Const(UInt8) -> 1 UInt8 : 6                                                  │
│            FUNCTION equals(d : 5, 1 :: 6) -> equals(d, 1) UInt8 : 7                            │
│   Positions: 0 1 2 3 4 5 7                                                                     │
│     Join (JOIN)                                                                                │
│       Expression (Before JOIN)                                                                 │
│       Actions: INPUT :: 0 -> a Int32 : 0                                                       │
│                INPUT :: 1 -> b Int32 : 1                                                       │
│                INPUT :: 2 -> c Int32 : 2                                                       │
│       Positions: 0 1 2                                                                         │
│         SettingQuotaAndLimits (Set limits and quota after reading from storage)                │
│           ReadFromMergeTree                                                                    │
│           ReadType: Default                                                                    │
│           Parts: 2                                                                             │
│           Granules: 2                                                                          │
│       Expression ((Joined actions + (Rename joined columns + (Projection + Before ORDER BY)))) │
│       Actions: INPUT : 0 -> d Int32 : 0                                                        │
│                INPUT : 1 -> a Int32 : 1                                                        │
│                INPUT : 2 -> b Int32 : 2                                                        │
│                ALIAS d :: 0 -> d Int32 : 3                                                     │
│                ALIAS a :: 1 -> b.a Int32 : 0                                                   │
│                ALIAS b :: 2 -> b.b Int32 : 1                                                   │
│       Positions: 1 0 3                                                                         │
│         SettingQuotaAndLimits (Set limits and quota after reading from storage)                │
│           ReadFromMergeTree                                                                    │
│           ReadType: Default                                                                    │
│           Parts: 2                                                                             │
│           Granules: 2                                                                          │
└────────────────────────────────────────────────────────────────────────────────────────────────┘

43 rows in set. Elapsed: 0.007 sec.

``` 

解释: 

  - Join (JOIN) -> JoinStep (其包含两个Step: query_plan 和 joined_plan, query_plan针对左表, joined_plan针对右表)

  - Expression (Before JOIN) -> ExpressionStep, 针对左表进行计算

  - Expression ((Joined actions + (Rename joined columns + (Projection + Before ORDER BY)))) -> ExpressionStep, 针对右表进行计算

  - Actions是ExpressionStep特有的属性, 规定了对一个block求值时的动作DAG
  - Filter column: equals(d, 1) (removed) : 过滤用的列为d=1, 如果结果集中不包括d=1, 在filter中就会将其移除. 否则 (比如 select d=1 ...), 此处不会出现removed
  - 执行计划的结构: 
    - Step可以包括子Step
    - 每一类Step有单独的属性, 比如ExpressionStep可以定义其对block操作的actions DAG

# 实验: JOIN谓词不下推

先进行 JOIN, 然后进行filter, 再进行order by

```
localhost :) explain actions=1 select * from a join b on a.b=b.b where a.a = 1 order by b.d;

EXPLAIN actions = 1
SELECT *
FROM a
INNER JOIN b ON a.b = b.b
WHERE a.a = 1
ORDER BY b.d ASC

Query id: 27a69d8d-86d5-445a-ade8-bb02897b43f8

┌─explain────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Expression (Projection)                                                                                │
│ Actions: INPUT :: 0 -> a Int32 : 0                                                                     │
│          INPUT :: 1 -> b Int32 : 1                                                                     │
│          INPUT :: 2 -> c Int32 : 2                                                                     │
│          INPUT :: 3 -> b.a Int32 : 3                                                                   │
│          INPUT :: 4 -> b.b Int32 : 4                                                                   │
│          INPUT :: 5 -> d Int32 : 5                                                                     │
│ Positions: 0 1 2 3 4 5                                                                                 │
│   MergingSorted (Merge sorted streams for ORDER BY)                                                    │
│   Sort description: d ASC                                                                              │
│     MergeSorting (Merge sorted blocks for ORDER BY)                                                    │
│     Sort description: d ASC                                                                            │
│       PartialSorting (Sort each block for ORDER BY)                                                    │
│       Sort description: d ASC                                                                          │
│         Expression (Before ORDER BY)                                                                   │
│         Actions: INPUT :: 0 -> a Int32 : 0                                                             │
│                  INPUT :: 1 -> b Int32 : 1                                                             │
│                  INPUT :: 2 -> c Int32 : 2                                                             │
│                  INPUT :: 3 -> b.a Int32 : 3                                                           │
│                  INPUT :: 4 -> b.b Int32 : 4                                                           │
│                  INPUT :: 5 -> d Int32 : 5                                                             │
│         Positions: 0 1 2 3 4 5                                                                         │
│           Filter (WHERE)                                                                               │
│           Filter column: equals(a, 1) (removed)                                                        │
│           Actions: INPUT : 0 -> a Int32 : 0                                                            │
│                    INPUT :: 1 -> b Int32 : 1                                                           │
│                    INPUT :: 2 -> c Int32 : 2                                                           │
│                    INPUT :: 3 -> b.a Int32 : 3                                                         │
│                    INPUT :: 4 -> b.b Int32 : 4                                                         │
│                    INPUT :: 5 -> d Int32 : 5                                                           │
│                    COLUMN Const(UInt8) -> 1 UInt8 : 6                                                  │
│                    FUNCTION equals(a : 0, 1 :: 6) -> equals(a, 1) UInt8 : 7                            │
│           Positions: 0 1 2 3 4 5 7                                                                     │
│             Join (JOIN)                                                                                │
│               Expression (Before JOIN)                                                                 │
│               Actions: INPUT :: 0 -> a Int32 : 0                                                       │
│                        INPUT :: 1 -> b Int32 : 1                                                       │
│                        INPUT :: 2 -> c Int32 : 2                                                       │
│               Positions: 0 1 2                                                                         │
│                 SettingQuotaAndLimits (Set limits and quota after reading from storage)                │
│                   ReadFromMergeTree                                                                    │
│                   ReadType: Default                                                                    │
│                   Parts: 1                                                                             │
│                   Granules: 1                                                                          │
│               Expression ((Joined actions + (Rename joined columns + (Projection + Before ORDER BY)))) │
│               Actions: INPUT : 0 -> a Int32 : 0                                                        │
│                        INPUT : 1 -> b Int32 : 1                                                        │
│                        INPUT : 2 -> d Int32 : 2                                                        │
│                        ALIAS a :: 0 -> b.a Int32 : 3                                                   │
│                        ALIAS b :: 1 -> b.b Int32 : 0                                                   │
│                        ALIAS d :: 2 -> d Int32 : 1                                                     │
│               Positions: 0 3 1                                                                         │
│                 SettingQuotaAndLimits (Set limits and quota after reading from storage)                │
│                   ReadFromMergeTree                                                                    │
│                   ReadType: Default                                                                    │
│                   Parts: 1                                                                             │
│                   Granules: 1                                                                          │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘

57 rows in set. Elapsed: 0.007 sec.

``` 

  - 先进行JOIN, 然后Filter, 最后Sorting

# 实验: IN与JOIN的不同

验证原因: "当两表关联查询只需要从左表出结果时，建议用IN而不是JOIN，即写成SELECT ... FROM left_table WHERE join_key IN (SELECT ... FROM right_table)的风格"

JOIN的执行计划:

```
localhost :) explain actions=1 select * from a join b on a.b = b.b where d = 1;

EXPLAIN actions = 1
SELECT *
FROM a
INNER JOIN b ON a.b = b.b
WHERE d = 1

Query id: 65f96916-96fb-4a9e-bc97-9795d3289157

┌─explain────────────────────────────────────────────────────────────────────────────────────────┐
│ Expression ((Projection + Before ORDER BY))                                                    │
│ Actions: INPUT :: 0 -> a Int32 : 0                                                             │
│          INPUT :: 1 -> b Int32 : 1                                                             │
│          INPUT :: 2 -> c Int32 : 2                                                             │
│          INPUT :: 3 -> b.a Int32 : 3                                                           │
│          INPUT :: 4 -> b.b Int32 : 4                                                           │
│          INPUT :: 5 -> d Int32 : 5                                                             │
│ Positions: 0 1 2 3 4 5                                                                         │
│   Filter (WHERE)                                                                               │
│   Filter column: equals(d, 1) (removed)                                                        │
│   Actions: INPUT :: 0 -> a Int32 : 0                                                           │
│            INPUT :: 1 -> b Int32 : 1                                                           │
│            INPUT :: 2 -> c Int32 : 2                                                           │
│            INPUT :: 3 -> b.a Int32 : 3                                                         │
│            INPUT :: 4 -> b.b Int32 : 4                                                         │
│            INPUT : 5 -> d Int32 : 5                                                            │
│            COLUMN Const(UInt8) -> 1 UInt8 : 6                                                  │
│            FUNCTION equals(d : 5, 1 :: 6) -> equals(d, 1) UInt8 : 7                            │
│   Positions: 0 1 2 3 4 5 7                                                                     │
│     Join (JOIN)                                                                                │
│       Expression (Before JOIN)                                                                 │
│       Actions: INPUT :: 0 -> a Int32 : 0                                                       │
│                INPUT :: 1 -> b Int32 : 1                                                       │
│                INPUT :: 2 -> c Int32 : 2                                                       │
│       Positions: 0 1 2                                                                         │
│         SettingQuotaAndLimits (Set limits and quota after reading from storage)                │
│           ReadFromMergeTree                                                                    │
│           ReadType: Default                                                                    │
│           Parts: 1                                                                             │
│           Granules: 1                                                                          │
│       Expression ((Joined actions + (Rename joined columns + (Projection + Before ORDER BY)))) │
│       Actions: INPUT : 0 -> d Int32 : 0                                                        │
│                INPUT : 1 -> a Int32 : 1                                                        │
│                INPUT : 2 -> b Int32 : 2                                                        │
│                ALIAS d :: 0 -> d Int32 : 3                                                     │
│                ALIAS a :: 1 -> b.a Int32 : 0                                                   │
│                ALIAS b :: 2 -> b.b Int32 : 1                                                   │
│       Positions: 1 0 3                                                                         │
│         SettingQuotaAndLimits (Set limits and quota after reading from storage)                │
│           ReadFromMergeTree                                                                    │
│           ReadType: Default                                                                    │
│           Parts: 1                                                                             │
│           Granules: 1                                                                          │
└────────────────────────────────────────────────────────────────────────────────────────────────┘

43 rows in set. Elapsed: 0.003 sec.

``` 

IN的执行计划:

```
localhost :) explain actions=1 select * from a where a.b in (select b from b where d = 1);

EXPLAIN actions = 1
SELECT *
FROM a
WHERE a.b IN
(
    SELECT b
    FROM b
    WHERE d = 1
)

Query id: 6b14321f-5e2f-41ec-9201-de5c7f65f5ea

┌─explain─────────────────────────────────────────────────────────────────────────┐
│ Expression (Projection)                                                         │
│ Actions: INPUT :: 0 -> b Int32 : 0                                              │
│          INPUT :: 1 -> a Int32 : 1                                              │
│          INPUT :: 2 -> c Int32 : 2                                              │
│ Positions: 1 0 2                                                                │
│   CreatingSets (Create sets before main query execution)                        │
│     Expression (Before ORDER BY)                                                │
│     Actions: INPUT :: 0 -> b Int32 : 0                                          │
│              INPUT :: 1 -> a Int32 : 1                                          │
│              INPUT :: 2 -> c Int32 : 2                                          │
│     Positions: 0 1 2                                                            │
│       SettingQuotaAndLimits (Set limits and quota after reading from storage)   │
│         ReadFromMergeTree                                                       │
│         ReadType: Default                                                       │
│         Parts: 1                                                                │
│         Granules: 1                                                             │
│     CreatingSet (Create set for subquery)                                       │
│     Set: _subquery2                                                             │
│       Expression ((Projection + Before ORDER BY))                               │
│       Actions: INPUT :: 0 -> b Int32 : 0                                        │
│       Positions: 0                                                              │
│         SettingQuotaAndLimits (Set limits and quota after reading from storage) │
│           ReadFromMergeTree                                                     │
│           ReadType: Default                                                     │
│           Parts: 1                                                              │
│           Granules: 1                                                           │
└─────────────────────────────────────────────────────────────────────────────────┘

26 rows in set. Elapsed: 0.003 sec.

``` 

性能测试: 

根据 <https://altinity.com/blog/2020/1/1/clickhouse-cost-efficiency-in-action-analyzing-500-billion-rows-on-an-intel-nuc>, 制造5亿数据

参考 <https://altinity.com/blog/2020/4/8/five-ways-to-handle-as-of-queries-in-clickhouse>, 进行JOIN和IN的速率对比: 

JOIN:

```
localhost :) SELECT *
:-] FROM readings
:-] INNER JOIN
:-] (
:-]     SELECT
:-]         sensor_id,
:-]         max(time) AS time
:-]     FROM readings
:-]     WHERE (sensor_id = 12345) AND (date <= '2019-08-01')
:-]     GROUP BY sensor_id
:-] ) AS last USING (sensor_id, time)
:-] WHERE sensor_id = 12345

SELECT *
FROM readings
INNER JOIN
(
    SELECT
        sensor_id,
        max(time) AS time
    FROM readings
    WHERE (sensor_id = 12345) AND (date <= '2019-08-01')
    GROUP BY sensor_id
) AS last USING (sensor_id, time)
WHERE sensor_id = 12345

Query id: 5e634763-b98a-4a66-96d4-d7cf266cd06a

┌─sensor_id─┬────────────────time─┬─temperature─┐
│     12345 │ 2019-01-01 23:59:00 │       55.81 │
└───────────┴─────────────────────┴─────────────┘

1 rows in set. Elapsed: 0.050 sec. Processed 16.38 thousand rows, 163.84 KB (328.51 thousand rows/s., 3.29 MB/s.)

``` 

IN:

```
localhost :) SELECT *
:-] FROM readings
:-] WHERE (sensor_id, time) IN
:-] (
:-]     SELECT
:-]         sensor_id,
:-]         max(time)
:-]     FROM readings
:-]     WHERE (sensor_id = 12345) AND (date <= '2019-08-01')
:-]     GROUP BY sensor_id
:-] ) and sensor_id = 12345

SELECT *
FROM readings
WHERE ((sensor_id, time) IN
(
    SELECT
        sensor_id,
        max(time)
    FROM readings
    WHERE (sensor_id = 12345) AND (date <= '2019-08-01')
    GROUP BY sensor_id
)) AND (sensor_id = 12345)

Query id: 77ebe96d-a690-4bbb-a36d-d250f9e08066

┌─sensor_id─┬────────────────time─┬─temperature─┐
│     12345 │ 2019-01-01 23:59:00 │       55.81 │
└───────────┴─────────────────────┴─────────────┘

1 rows in set. Elapsed: 0.013 sec. Processed 8.19 thousand rows, 71.94 KB (620.26 thousand rows/s., 5.45 MB/s.)

``` 

IN 比 JOIN处理的行数少一半, 观察IN的执行计划: 

```
localhost :) SELECT *
:-] FROM readings
:-] WHERE (sensor_id, time) IN
:-] (
:-]     SELECT
:-]         sensor_id,
:-]         max(time)
:-]     FROM readings
:-]     WHERE (sensor_id = 12345) AND (date <= '2019-08-01')
:-]     GROUP BY sensor_id
:-] ) and sensor_id = 12345

SELECT *
FROM readings
WHERE ((sensor_id, time) IN
(
    SELECT
        sensor_id,
        max(time)
    FROM readings
    WHERE (sensor_id = 12345) AND (date <= '2019-08-01')
    GROUP BY sensor_id
)) AND (sensor_id = 12345)

Query id: 77ebe96d-a690-4bbb-a36d-d250f9e08066

┌─sensor_id─┬────────────────time─┬─temperature─┐
│     12345 │ 2019-01-01 23:59:00 │       55.81 │
└───────────┴─────────────────────┴─────────────┘

1 rows in set. Elapsed: 0.013 sec. Processed 8.19 thousand rows, 71.94 KB (620.26 thousand rows/s., 5.45 MB/s.)

``` 

可以看到IN进行了同表优化, 仅读一次表. 而JOIN则会如实的读两次表, 并进行JOIN操作. 

# 使用新版本的tpcds

```
下载 tpc-ds-tool.v3.1.0rc2
 
apt-get install bison flex
 
//需要使用改写后的表结构, 参看其他tpc-ds的实验
 
cd tools/; make
 
./tools# ./dsdgen -sc 1 -dir ../data -TERMINATE N
 
for f in *.dat
do
echo $f
clickhouse-client --input_format_with_names_use_header=0 --host=127.0.0.1 -d tpcds --password clickhouse --format_csv_delimiter="|" --query="INSERT INTO "${f%.dat}" FORMAT CSV" < $f
done
 
//仅有dbgen_version.dat会报错, 应该是改写的表结构不匹配, 此处忽略
``` 

# 实验: 测试两个不同表的IN和JOIN的效率差异

JOIN:

```
localhost :) select distinct(w_warehouse_name) from warehouse join inventory on inv_warehouse_sk = w_warehouse_sk where inv_quantity_on_hand > 900;

SELECT DISTINCT w_warehouse_name
FROM warehouse
INNER JOIN inventory ON inv_warehouse_sk = w_warehouse_sk
WHERE inv_quantity_on_hand > 900

Query id: 1f58510d-2a06-4f3e-a5e2-c18fc6ea625f

┌─w_warehouse_name─────┐
│ Conventional childr  │
│ Important issues liv │
│ Doors canno          │
│ Bad cards must make. │
│                      │
└──────────────────────┘

5 rows in set. Elapsed: 0.354 sec. Processed 11.75 million rows, 93.96 MB (33.13 million rows/s., 265.07 MB/s.)

``` 

将过滤条件下推: 

```
localhost :) select distinct(w_warehouse_name) from warehouse join (select * from inventory where inv_quantity_on_hand > 900) b on inv_warehouse_sk = w_warehouse_sk;

SELECT DISTINCT w_warehouse_name
FROM warehouse
INNER JOIN
(
    SELECT *
    FROM inventory
    WHERE inv_quantity_on_hand > 900
) AS b ON inv_warehouse_sk = w_warehouse_sk

Query id: b422ce0e-7c12-4880-ac34-d49b13e03a5f

┌─w_warehouse_name─────┐
│ Conventional childr  │
│ Important issues liv │
│ Doors canno          │
│ Bad cards must make. │
│                      │
└──────────────────────┘

5 rows in set. Elapsed: 0.217 sec. Processed 11.75 million rows, 93.96 MB (54.02 million rows/s., 432.18 MB/s.)

``` 

IN: 

```
localhost :) select w_warehouse_name from warehouse where w_warehouse_sk in (select inv_warehouse_sk from inventory where inv_quantity_on_hand > 900);

SELECT w_warehouse_name
FROM warehouse
WHERE w_warehouse_sk IN
(
    SELECT inv_warehouse_sk
    FROM inventory
    WHERE inv_quantity_on_hand > 900
)

Query id: 4da7509f-f8c0-4560-abdd-beaceffd2983

┌─w_warehouse_name─────┐
│ Conventional childr  │
│ Important issues liv │
│ Doors canno          │
│ Bad cards must make. │
│                      │
└──────────────────────┘

5 rows in set. Elapsed: 0.047 sec.

``` 

性能差异的原因 未知

# 问题总结描述

1\. JOIN -> 0.369s  
select distinct(w_warehouse_name) from warehouse  
join inventory  
on inv_warehouse_sk = w_warehouse_sk  
where inv_quantity_on_hand > 900;

  
2\. IN -> 0.033s  
select w_warehouse_name from warehouse where w_warehouse_sk in  
(select inv_warehouse_sk from inventory where inv_quantity_on_hand > 900);

  
3\. 条件下推的JOIN -> 0.209s  
select distinct(w_warehouse_name) from warehouse join  
(select inv_warehouse_sk from inventory where inv_quantity_on_hand > 900) b on inv_warehouse_sk = w_warehouse_sk;

  
4\. 条件下推+distinct子句的JOIN -> 0.121s  
select distinct(w_warehouse_name) from  
warehouse join  
(select distinct(inv_warehouse_sk) from inventory where inv_quantity_on_hand > 900) b  
on inv_warehouse_sk = w_warehouse_sk;

  
5\. 对比: 右表换成system.numbers -> 0.004s  
select w_warehouse_name from  
warehouse join  
(select toInt32(number) as inv_warehouse_sk from system.numbers limit 5 offset 1) b  
on inv_warehouse_sk = w_warehouse_sk;

  
6\. 左右表对调 -> 0.046s  
select distinct(w_warehouse_name) from  
(select (inv_warehouse_sk) from inventory where inv_quantity_on_hand > 900) b join  
warehouse  
on inv_warehouse_sk = w_warehouse_sk;

# 理解JOIN的实现

左右表对调导致性能不同, 涉及到JOIN的实现原理, 需要进行了解

  - <https://zhuanlan.zhihu.com/p/377506070>
    - ClickHouse 的 HASH JOIN算法实现比较简单：

      - 从right_table 读取该表全量数据，在内存中构建HASH MAP；

      - 从left_table 分批读取数据，根据JOIN KEY到HASH MAP中进行查找，如果命中，则该数据作为JOIN的输出；

    - ![image](https://pic2.zhimg.com/80/v2-0075207bf7bb008827effb593d21a385_1440w.jpg)

  - 代码相关
    - 接口 IJoin
      - addJoinedBlock 为第一阶段: 提取右表的全量数据, 构建HASH MAP
      - joinBlock 为第二阶段: 分批读取左表数据, 于HASH MAP中查找, 有命中的记录, 则与右表JOIN后输出
        - 其中调用了joinRightColumns, 包含主要逻辑

# 研究左右表对调的差异

原始SQL的执行计划: 

```
localhost :) explain pipeline graph=1 select distinct(w_warehouse_name) from warehouse join
:-] (select inv_warehouse_sk from inventory where inv_quantity_on_hand > 900) b on inv_warehouse_sk = w_warehouse_sk

EXPLAIN PIPELINE graph = 1
SELECT DISTINCT w_warehouse_name
FROM warehouse
INNER JOIN
(
    SELECT inv_warehouse_sk
    FROM inventory
    WHERE inv_quantity_on_hand > 900
) AS b ON inv_warehouse_sk = w_warehouse_sk

Query id: 9d5827bf-3762-4f3e-bdb7-e368bf8c61c3

┌─explain───────────────────────────────────────┐
│ digraph                                       │
│ {                                             │
│   rankdir="LR";                               │
│   { node [shape = box]                        │
│         n7 [label="FillingRightJoinSide"];    │
│         n3 [label="JoiningTransform"];        │
│         n1 [label="MergeTree"];               │
│         n4 [label="MergeTreeThread × 8"];     │
│         n6 [label="Resize"];                  │
│     subgraph cluster_0 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n2 [label="ExpressionTransform"];     │
│       }                                       │
│     }                                         │
│     subgraph cluster_1 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n8 [label="ExpressionTransform"];     │
│       }                                       │
│     }                                         │
│     subgraph cluster_2 {                      │
│       label ="Distinct";                      │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n9 [label="DistinctTransform"];       │
│       }                                       │
│     }                                         │
│     subgraph cluster_3 {                      │
│       label ="Distinct";                      │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n10 [label="DistinctTransform"];      │
│       }                                       │
│     }                                         │
│     subgraph cluster_4 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n5 [label="ExpressionTransform × 8"]; │
│       }                                       │
│     }                                         │
│     subgraph cluster_5 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n11 [label="ExpressionTransform"];    │
│       }                                       │
│     }                                         │
│   }                                           │
│   n7 -> n3 [label=""];                        │
│   n3 -> n8 [label=""];                        │
│   n1 -> n2 [label=""];                        │
│   n4 -> n5 [label="× 8"];                     │
│   n6 -> n7 [label=""];                        │
│   n2 -> n3 [label=""];                        │
│   n8 -> n9 [label=""];                        │
│   n9 -> n10 [label=""];                       │
│   n10 -> n11 [label=""];                      │
│   n5 -> n6 [label="× 8"];                     │
│ }                                             │
└───────────────────────────────────────────────┘

75 rows in set. Elapsed: 0.006 sec.

``` 

![QQ20210608-182839@2x.png](/assets/01KJBYDBWTJGYAJWBZ93943E5R/QQ20210608-182839%402x.png)

左右表对调的执行计划: 

```
localhost :) explain pipeline graph=1 select distinct(w_warehouse_name) from
:-] (select (inv_warehouse_sk) from inventory where inv_quantity_on_hand > 900) b join
:-] warehouse on inv_warehouse_sk = w_warehouse_sk

EXPLAIN PIPELINE graph = 1
SELECT DISTINCT w_warehouse_name
FROM
(
    SELECT inv_warehouse_sk
    FROM inventory
    WHERE inv_quantity_on_hand > 900
) AS b
INNER JOIN warehouse ON inv_warehouse_sk = w_warehouse_sk

Query id: fb2efcfd-5444-4a0e-ba78-05b24e11b7fa

┌─explain───────────────────────────────────────┐
│ digraph                                       │
│ {                                             │
│   rankdir="LR";                               │
│   { node [shape = box]                        │
│         n6 [label="FillingRightJoinSide"];    │
│         n3 [label="JoiningTransform × 8"];    │
│         n4 [label="MergeTree"];               │
│         n1 [label="MergeTreeThread × 8"];     │
│         n7 [label="Resize"];                  │
│     subgraph cluster_0 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n5 [label="ExpressionTransform"];     │
│       }                                       │
│     }                                         │
│     subgraph cluster_1 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n2 [label="ExpressionTransform × 8"]; │
│       }                                       │
│     }                                         │
│     subgraph cluster_2 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n8 [label="ExpressionTransform × 8"]; │
│       }                                       │
│     }                                         │
│     subgraph cluster_3 {                      │
│       label ="Distinct";                      │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n9 [label="DistinctTransform × 8"];   │
│       }                                       │
│     }                                         │
│     subgraph cluster_4 {                      │
│       label ="Distinct";                      │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n11 [label="DistinctTransform"];      │
│         n10 [label="Resize"];                 │
│       }                                       │
│     }                                         │
│     subgraph cluster_5 {                      │
│       label ="Expression";                    │
│       style=filled;                           │
│       color=lightgrey;                        │
│       node [style=filled,color=white];        │
│       { rank = same;                          │
│         n12 [label="ExpressionTransform"];    │
│       }                                       │
│     }                                         │
│   }                                           │
│   n6 -> n7 [label=""];                        │
│   n3 -> n8 [label="× 8"];                     │
│   n4 -> n5 [label=""];                        │
│   n1 -> n2 [label="× 8"];                     │
│   n7 -> n3 [label="× 8"];                     │
│   n5 -> n6 [label=""];                        │
│   n2 -> n3 [label="× 8"];                     │
│   n8 -> n9 [label="× 8"];                     │
│   n9 -> n10 [label="× 8"];                    │
│   n11 -> n12 [label=""];                      │
│   n10 -> n11 [label=""];                      │
│ }                                             │
└───────────────────────────────────────────────┘

77 rows in set. Elapsed: 0.008 sec.
``` 

![QQ20210608-183026@2x.png](/assets/01KJBYDBWTJGYAJWBZ93943E5R/QQ20210608-183026%402x.png)

结论: 

  1. 在JOIN前, clickhouse必须完全计算出右表内容, 所以右表应放 小表/计算成本低的表
  2. JOIN以及JOIN后的计算, 可以和左表的获取并行, 所以应将计算量大且可以并发计算的表, 放在JOIN的左表

设置join_algorithm=partial_merge测试, 结果类似.

# Merge Join

Merge Join的接口也是IJoin, 与Hash Join一致, 也就是说其也要先准备好right table, 然后才对left table分块进行join

对left table和right table的处理, 都是先将其排序, 再对两表进行合并

参数: 

  - partial_merge_join_rows_in_right_blocks
    - 在内存中能容下的右表大小, 超过大小将转移到磁盘上
  - join_on_disk_max_files_to_merge
    - 对于在磁盘上进行合并, 同时允许多少个文件合并

优化参数: 

  - partial_merge_join_optimizations
  - partial_merge_join_left_table_buffer_bytes

当不进行优化时, left table以block为单位进行处理; 当开启优化时, clickhouse准备一个buffer, 其大小如上参数控制, 积累多个block, 统一进行排序处理, 使得效率增加

Partial merge的来源: 

根据[20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation], partial merge定义了数据交换的形式, 如图: 

![image2021-6-13 14:53:58.png](/assets/01KJBYDBWTJGYAJWBZ93943E5R/image2021-6-13%2014%3A53%3A58.png)

# Join Engine

TODO: 实验

Join Engine 实现了IStorage接口:

  - 重点是write接口 (SetOrJoinBlockOutputStream::write), 其中
    - 调用了StorageJoin::insertBlock, 将数据放入内存中的Join的右表结构 (HashJoin / MergeJoin)
    - 对数据进行持久化备份

Join Engine的特点:

  - 记录了Join的右表结构, 不需要每次构造 Hash表 或者 Merge中的排序右表
  - 允许内存结构进行持久化, 在重启后继续使用
