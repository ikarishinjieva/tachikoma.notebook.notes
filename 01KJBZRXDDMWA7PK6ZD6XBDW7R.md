---
title: 20250306 - 为SQL优化的模型微调方案, 找到合适的校验集
confluence_page_id: 3801147
created_at: 2025-03-06T12:01:02+00:00
updated_at: 2025-03-11T02:24:00+00:00
---

# 找到业务实际发生的SQL验证数据集

<https://github.com/glee4810/EHRSQL>

  - 其是根据一篇论文构建的数据集, 可以从论文方向寻找数据集

<https://github.com/awslabs/unified-text2sql-benchmark>

  - 大杂烩, 其数据集列表: <https://github.com/awslabs/unified-text2sql-benchmark/blob/main/original/download.sh>

<https://github.com/peterbaile/beaver>

  - 论文

<https://github.com/grandchicken/MultiSQL/tree/main>

  - 论文

<https://sig4kg.github.io/archer-bench/>

  - 论文

<https://github.com/defog-ai/sql-eval/tree/main/data>

  - <https://paperswithcode.com/>

<https://huggingface.co/datasets/Exploration-Lab/BookSQL?row=97>

  - 论文

<https://github.com/hirupert/sede/blob/main/data/sede/train.jsonl>

  - 论文

# 对SQL进行聚类采样

方案: 对SQL提取关键字, 用关键字序列的带权编辑距离作为空间距离度量

脚本: [test.py](/assets/01KJBZRXDDMWA7PK6ZD6XBDW7R/test.py)

数据集: <https://github.com/glee4810/EHRSQL>

进行聚类的结果: [sql.cluster.log](/assets/01KJBZRXDDMWA7PK6ZD6XBDW7R/sql.cluster.log)

发现的问题:

  - 聚类速度很慢, 需要优化算法

对算法进行优化, 大幅缩短时间: [test.2.py](/assets/01KJBZRXDDMWA7PK6ZD6XBDW7R/test.2.py)

尝试集中算法后, 明确需要考虑的方向: 

  - 使用编辑距离进行聚类, 会发现需要构建完整的距离矩阵 (编辑距离不具有传递性, 所以两两间的距离都需要计算一次). 完整的距离矩阵耗时长, 且需要大量内存
  - 使用 Tf-idf 这一类简单的聚类算法, 其对关键字的顺序没有认知
  - 使用 MinHash + LSH 优化 (一种类hash算法), 可以增加对关键字的顺序的认知, 但其是靠阈值来划分聚类, 阈值不方便取, 需要多次尝试

最终使用的方案是两阶段聚类:

  - 阶段一: 使用Tf-idf来进行初步聚类, 以降低 阶段二 的节点数, 这样能解决耗时和内存问题, 牺牲了部分精度
  - 阶段二: 使用编辑距离 构建完整矩阵, 进行精确的聚类

这种方案的精度和速度都大表

脚本: [test.3.ipynb](/assets/01KJBZRXDDMWA7PK6ZD6XBDW7R/test.3.ipynb)

# 如何根据规则构建"错误的"SQL

已经获取了比较好的SQL空间, 现在需要用这个SQL空间 变化出 违反规则的SQL空间

尝试: 

```
我有一个SQL需要遵守的规则:

{

"规则ID": "rule0001",

"规则名称": "投影下推",

"规则描述": "通过删除子查询中无意义的列, 这些列在外查询中没有被使用, 来减少IO和网络的代价, 同时提升优化器在进行表访问的规划时, 采用无需回表的优化选项的几率。",

}

我有一个SQL, 我希望你对这个SQL进行改写, 让改写后的SQL能违反这个规则. 你能列出一些明确的改写手段, 让改写后的SQL违反这个规则么
 
------------------------------
 
为了构造违反"投影下推"规则的SQL，可以通过以下手段进行改写：

无意义列追加法

在子查询中保留甚至添加外层查询不需要的列：

-- 原合规SQL
SELECT dept_name 
FROM (
    SELECT department_id, dept_name  -- 仅保留必要列
    FROM departments
) subq
WHERE department_id > 100;

-- 改写为违规SQL
SELECT dept_name 
FROM (
    SELECT *, location_id  -- 添加无关列（假设location_id未被使用）
    FROM departments
) subq
WHERE department_id > 100;

星号展开法

使用通配符保留所有列：

-- 原合规SQL
SELECT employee_id 
FROM (
    SELECT employee_id, hire_date 
    FROM employees
) subq
WHERE hire_date > '2020-01-01';

-- 改写为违规SQL
SELECT employee_id 
FROM (
    SELECT *  -- 保留所有列
    FROM employees
) subq
WHERE hire_date > '2020-01-01';

冗余计算列法

添加无实际用途的计算字段：

-- 原合规SQL
SELECT product_name 
FROM (
    SELECT product_id, product_name 
    FROM products
) subq
WHERE product_id < 1000;

-- 改写为违规SQL
SELECT product_name 
FROM (
    SELECT product_id, 
           product_name,
           unit_price * 0  -- 无效计算列
    FROM products
) subq
WHERE product_id < 1000;

多级嵌套保留法

通过多层嵌套保留冗余列：

-- 原合规SQL
SELECT customer_name 
FROM (
    SELECT cust_id, customer_name 
    FROM customers
) subq1;

-- 改写为违规SQL
SELECT customer_name 
FROM (
    SELECT cust_id, customer_name, phone_number 
    FROM (
        SELECT *  -- 第一层保留所有列
        FROM customers
    ) subq2
) subq1;

虚假列引用法

构造看似有用实则无用的列引用：

-- 原合规SQL
SELECT order_date 
FROM (
    SELECT order_id, order_date 
    FROM orders
) subq
WHERE order_id BETWEEN 100 AND 200;

-- 改写为违规SQL
SELECT order_date 
FROM (
    SELECT order_id, 
           order_date,
           CASE WHEN customer_id > 0 THEN 1 ELSE 0 END AS dummy_flag  -- 未被使用的判断列
    FROM orders
) subq
WHERE order_id BETWEEN 100 AND 200;

这些改写手段的共同特征：

在子查询中保留未在父查询中使用的列

增加冗余计算字段或全列引用

通过多层嵌套传播冗余列

制造"看似有用实则无用"的列引用假象

通过以上方法改写后的SQL，将无法进行有效的投影下推优化，导致：

增加存储引擎的IO量（尤其当使用行式存储时）

增大网络传输数据量（分布式数据库场景更明显）

降低使用覆盖索引的概率

增加内存消耗（需要缓存更多冗余字段）

``` 

# 使用代码对孙健的训练模型进行测试

<https://github.com/ikarishinjieva/sql-ai>

commit: ab535ae292ac8ed4ea1e07ac2332b56d5112f486

脚本: 

  - 1_sql_cluster.ipynb
    - 从原始数据中读取SQL, 并进行聚类, 抽取聚类中心
      - 其中去除了sede和BookSQL的数据. sede的SQL是多SQL聚合, 目前不考虑. BookSQL的体量比较大且特征集中, 可以后续加入.
    - 使用两阶段聚类
  - 2_add_fault_in_sql.ipynb
    - 对SQL注入某种问题, 最大改写次数是3次
      - 目前使用的改写策略仅为: 无意义列追加法
      - 可以扩展策略
  - 3_evaluate.ipynb
    - 使用孙健的训练模型进行评估
    - 其中使用并发, 两个GPU进行并发评估
    - 评估方法是: 检测片段是否在改写后的SQL中消失, 不做其他检测
