---
title: 20250319 - 研究SQL优化模型微调 - 靠结构流程影响优化流程
confluence_page_id: 3801223
created_at: 2025-03-19T08:53:33+00:00
updated_at: 2025-03-21T05:35:40+00:00
---

# 前继

[20250311 - 找到SQL优化模型训练的不同的训练方法]

# 额外: 使用4 GPU + DDP: 

参数: 

```
CUDA_VISIBLE_DEVICES=0,1,2,3 PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
NPROC_PER_NODE=2 \
swift sft \
    --model /opt/huangyan/sql-ai/checkpoints/DeepSeek-R1-Distill-Qwen-7B \
    --tuner_backend peft \
    --train_type lora \
    --torch_dtype bfloat16 \
    --dataset /opt/huangyan/sql-ai/dataset/sql_story.train.jsonl /opt/huangyan/sql-ai/dataset/sql_optimize.train.jsonl  /opt/huangyan/sql-ai/dataset/add.train.jsonl\
    --val_dataset /opt/huangyan/sql-ai/dataset/sql_story.eval.jsonl \
    --num_train_epochs 8 \
    --per_device_train_batch_size 1 \
    --learning_rate 1e-4 \
    --lora_rank 8 \
    --lora_alpha 32 \
    --gradient_accumulation_steps 1 \
    --eval_strategy epoch \
    --save_strategy epoch \
    --gradient_checkpointing_kwargs '{"use_reentrant": false}' \
    --logging_steps 5 >sft.1.log 2>&1

``` 

必要参数: 

  - NPROC_PER_NODE
  - \--gradient_checkpointing_kwargs '{"use_reentrant": false}' \

# 当前目标

训练1: 使用"结构流程"和"优化流程"两套数据进行微调, 让模型认知到两者的关联

验证1: 验证对于一个没见过的SQL A, 模型给出的"优化流程"是不具备"结构流程"特点的

SQL A: 

```
WITH ShipmentCTE AS (
    SELECT 
        o.order_id,
        o.client_id,
        c.client_name,
        o.origin_port_id,
        o.destination_port_id,
        o.cargo_weight,
        o.booking_date,
        s.shipment_id,
        s.vessel_id,
        v.vessel_name,
        s.departure_date,
        s.estimated_arrival,
        s.actual_arrival,
        CASE 
            WHEN s.actual_arrival > s.estimated_arrival THEN 1
            ELSE 0
        END AS delay_flag,
        DATEDIFF(day, s.estimated_arrival, COALESCE(s.actual_arrival, GETDATE())) AS delay_days,
        ROW_NUMBER() OVER(PARTITION BY o.client_id ORDER BY o.booking_date DESC) AS shipment_rank
    FROM 
        orders o
        INNER JOIN shipments s ON o.order_id = s.order_id
        INNER JOIN clients c ON o.client_id = c.client_id
        LEFT JOIN vessels v ON s.vessel_id = v.vessel_id
    WHERE 
        o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'
        AND o.cargo_type = 'CONTAINER'
),
PortStatistics AS (
    SELECT
        p.port_id,
        p.port_name,
        COUNT(DISTINCT s.shipment_id) AS total_shipments,
        SUM(s.cargo_weight) AS total_cargo,
        AVG(s.delay_days) AS avg_delay,
        COUNT(CASE WHEN s.delay_flag = 1 THEN 1 END) AS delayed_shipments,
        RANK() OVER(ORDER BY SUM(s.cargo_weight) DESC) AS cargo_rank
    FROM 
        ports p
        LEFT JOIN ShipmentCTE s ON p.port_id = s.origin_port_id
    GROUP BY 
        p.port_id, p.port_name
),
InventoryAnalysis AS (
    SELECT
        w.warehouse_id,
        w.location_code,
        i.sku,
        SUM(i.current_stock) AS total_inventory,
        COUNT(DISTINCT s.shipment_id) AS outgoing_shipments,
        SUM(s.cargo_weight) AS outgoing_weight,
        LEAD(SUM(s.cargo_weight)) OVER(PARTITION BY w.warehouse_id ORDER BY s.departure_date) AS next_shipment_weight
    FROM
        warehouses w
        INNER JOIN inventory i ON w.warehouse_id = i.warehouse_id
        LEFT JOIN ShipmentCTE s ON w.warehouse_id = s.origin_port_id
    GROUP BY
        w.warehouse_id, w.location_code, i.sku
)
SELECT 
    ps.port_name AS origin_port,
    ps.total_shipments,
    ps.total_cargo,
    ps.avg_delay,
    ps.delayed_shipments,
    ia.location_code,
    ia.outgoing_shipments,
    ia.outgoing_weight,
    (ia.total_inventory - ia.outgoing_weight) AS projected_inventory,
    cte.client_name,
    cte.vessel_name,
    cte.delay_days,
    CASE 
        WHEN cte.delay_days > 7 THEN 'Severe Delay'
        WHEN cte.delay_days > 3 THEN 'Moderate Delay'
        ELSE 'On Time'
    END AS delay_status
FROM 
    PortStatistics ps
    INNER JOIN ShipmentCTE cte ON ps.port_id = cte.origin_port_id
    LEFT JOIN InventoryAnalysis ia ON cte.origin_port_id = ia.warehouse_id
WHERE
    ps.cargo_rank <= 10
    AND cte.shipment_rank = 1
    AND ia.projected_inventory > 0
ORDER BY
    ps.total_cargo DESC,
    cte.departure_date ASC;

``` 

"优化流程":

(解释成了港口业务)

````
# 按照SQL结构进行分析

- **计算每个订单的运输详细信息（ShipmentCTE的构成）**  
  用于获取订单与运输相关的详细信息，包括延迟情况  
  ```sql
  SELECT 
      o.order_id,
      o.client_id,
      c.client_name,
      o.origin_port_id,
      o.destination_port_id,
      o.cargo_weight,
      o.booking_date,
      s.shipment_id,
      s.vessel_id,
      v.vessel_name,
      s.departure_date,
      s.estimated_arrival,
      s.actual_arrival,
      CASE 
          WHEN s.actual_arrival > s.estimated_arrival THEN 1
          ELSE 0
      END AS delay_flag,
      DATEDIFF(day, s.estimated_arrival, COALESCE(s.actual_arrival, GETDATE())) AS delay_days,
      ROW_NUMBER() OVER(PARTITION BY o.client_id ORDER BY o.booking_date DESC) AS shipment_rank
  FROM 
      orders o
      INNER JOIN shipments s ON o.order_id = s.order_id
      INNER JOIN clients c ON o.client_id = c.client_id
      LEFT JOIN vessels v ON s.vessel_id = v.vessel_id
  WHERE 
      o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'
      AND o.cargo_type = 'CONTAINER'
  ```

- **计算港口的运输统计信息（PortStatistics的构成）**  
  - **计算港口的总运输量**：统计每个港口的总运输量  
    ```sql
    COUNT(DISTINCT s.shipment_id) AS total_shipments
    ```
  - **计算港口的总货量**：统计每个港口的总货量  
    ```sql
    SUM(s.cargo_weight) AS total_cargo
    ```
  - **计算港口的平均延迟天数**：计算每个港口的平均延迟天数  
    ```sql
    AVG(s.delay_days) AS avg_delay
    ```
  - **计算延迟运输的数量**：统计每个港口的延迟运输数量  
    ```sql
    COUNT(CASE WHEN s.delay_flag = 1 THEN 1 END) AS delayed_shipments
    ```
  - **计算港口的货量排名**：计算每个港口的货量排名  
    ```sql
    RANK() OVER(ORDER BY SUM(s.cargo_weight) DESC) AS cargo_rank
    ```
  **构成**：通过`LEFT JOIN`连接`ports p`和`shipmentCTE s`，按`port_id`和`port_name`分组  
  ```sql
  SELECT 
      p.port_id,
      p.port_name,
      COUNT(DISTINCT s.shipment_id) AS total_shipments,
      SUM(s.cargo_weight) AS total_cargo,
      AVG(s.delay_days) AS avg_delay,
      COUNT(CASE WHEN s.delay_flag = 1 THEN 1 END) AS delayed_shipments,
      RANK() OVER(ORDER BY SUM(s.cargo_weight) DESC) AS cargo_rank
  FROM 
      ports p
      LEFT JOIN ShipmentCTE s ON p.port_id = s.origin_port_id
  GROUP BY 
      p.port_id, p.port_name
  ```

- **计算仓库的库存分析（InventoryAnalysis的构成）**  
  - **计算仓库的总库存**：统计每个仓库的总库存  
    ```sql
    SUM(i.current_stock) AS total_inventory
    ```
  - **计算仓库的出货量**：统计每个仓库的出货量  
    ```sql
    COUNT(DISTINCT s.shipment_id) AS outgoing_shipments
    ```
  - **计算仓库的出货重量**：统计每个仓库的出货重量  
    ```sql
    SUM(s.cargo_weight) AS outgoing_weight
    ```
  - **计算下一批货物的预期重量**：计算每个仓库的下一批货物的预期重量  
    ```sql
    LEAD(SUM(s.cargo_weight)) OVER(PARTITION BY w.warehouse_id ORDER BY s.departure_date) AS next_shipment_weight
    ```
  **构成**：通过`LEFT JOIN`连接`warehouses w`和`shipmentCTE s`，按`warehouse_id`、`location_code`和`sku`分组  
  ```sql
  SELECT 
      w.warehouse_id,
      w.location_code,
      i.sku,
      SUM(i.current_stock) AS total_inventory,
      COUNT(DISTINCT s.shipment_id) AS outgoing_shipments,
      SUM(s.cargo_weight) AS outgoing_weight,
      LEAD(SUM(s.cargo_weight)) OVER(PARTITION BY w.warehouse_id ORDER BY s.departure_date) AS next_shipment_weight
  FROM 
      warehouses w
      INNER JOIN inventory i ON w.warehouse_id = i.warehouse_id
      LEFT JOIN ShipmentCTE s ON w.warehouse_id = s.origin_port_id
  GROUP BY
      w.warehouse_id, w.location_code, i.sku
  ```

- **最终输出（最外层查询）**  
  将PortStatistics、InventoryAnalysis和CTE的结果合并，计算延迟状态并排序  
  ```sql
  SELECT 
      ps.port_name AS origin_port,
      ps.total_shipments,
      ps.total_cargo,
      ps.avg_delay,
      ps.delayed_shipments,
      ia.location_code,
      ia.outgoing_shipments,
      ia.outgoing_weight,
      (ia.total_inventory - ia.outgoing_weight) AS projected_inventory,
      cte.client_name,
      cte.vessel_name,
      cte.delay_days,
      CASE 
          WHEN cte.delay_days > 7 THEN 'Severe Delay'
          WHEN cte.delay_days > 3 THEN 'Moderate Delay'
          ELSE 'On Time'
      END AS delay_status
  FROM 
      PortStatistics ps
      INNER JOIN ShipmentCTE cte ON ps.port_id = cte.origin_port_id
      LEFT JOIN InventoryAnalysis ia ON cte.origin_port_id = ia.warehouse_id
  WHERE
      ps.cargo_rank <= 10
      AND cte.shipment_rank = 1
      AND ia.projected_inventory > 0
  ORDER BY
      ps.total_cargo DESC,
      cte.departure_date ASC
  ```

```` 

训练2: 在训练1的基础上, 仅补充SQL A的"结构流程"

将"结构流程"构建成"我的世界"的业务: 

````
# 按照SQL结构进行分析

- **激活末影箱网络：追踪所有集装箱运输记录（ShipmentCTE 基础数据层）**  
  末影箱自动捕获维度传送日志的魔法核心  
  ```sql
  SELECT 
    o.order_id,  -- 末影箱编号
    c.client_name,  -- 村民交易者名称
    v.vessel_name AS 末影船,  -- 绑定末影珍珠的运输船
    CASE  -- 红石比较器检测时间差
        WHEN s.actual_arrival > s.estimated_arrival THEN 1 
        ELSE 0 
    END AS 末地烛延迟信号,
    ROW_NUMBER() OVER(...) AS 传送优先级  -- 按村民交易时间倒序标记末影箱存取顺序
  FROM 
    orders o 
    INNER JOIN shipments s ...  -- 末影箱与传送门实体连接
    LEFT JOIN vessels v ...  -- 绑定末影船数据（允许空值）
  WHERE 
    o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'  -- 游戏年时间过滤器
    AND o.cargo_type = 'CONTAINER'  -- 仅处理集装箱型末影箱
  ```
  
  - **建造村民交易所：港口吞吐统计塔（PortStatistics 聚合层）**  
  自动生成村庄贸易报告的红石装置  
  ```sql
  SELECT
    p.port_name AS 港口名称,  -- 村庄名称
    COUNT(DISTINCT s.shipment_id) AS 末影珍珠投射次数,  -- 末影箱使用频次
    SUM(s.cargo_weight) AS 铁锭总重量,  -- 村民交易商品总重量
    RANK() OVER(ORDER BY SUM(...) DESC) AS 红石能量排名  -- 按货运量生成红石信号强度排序
  FROM 
    ports p 
    LEFT JOIN ShipmentCTE s ...  -- 连接末影箱网络数据
  GROUP BY 
    p.port_id  -- 按每个村庄独立统计
  ```
  
  - **组装红石仓库监控器（InventoryAnalysis 库存维度）**  
  带预测功能的投掷器库存管理系统  
  ```sql
  SELECT
    w.location_code AS 箱体坐标,  -- 仓库区块坐标
    LEAD(...) OVER(...) AS 下个脉冲信号强度,  -- 红石中继器预测下次发射强度
    SUM(...) - SUM(...) AS 剩余红石粉量  -- 当前库存减去已发射的红石信号
  FROM 
    warehouses w 
    INNER JOIN inventory i ...  -- 连接箱子与物品栏
    LEFT JOIN ShipmentCTE s ...  -- 获取末影箱发射记录
  GROUP BY 
    w.warehouse_id  -- 按每个仓库独立计算
  ```
  
  - **启动红石控制塔（最终主查询）**  
  整合所有信号的全自动指挥系统  
  ```sql
  SELECT 
    ps.port_name AS 主城名称,  -- 前十强贸易港
    ia.location_code AS 战略仓库坐标, 
    CASE  -- 红石灯延迟警报系统
        WHEN cte.delay_days > 7 THEN '凋灵警报' 
        WHEN cte.delay_days > 3 THEN '僵尸围城警报'
        ELSE '红石脉冲正常' 
    END AS 末影信号状态
  FROM 
    PortStatistics ps 
    INNER JOIN ShipmentCTE cte ...  -- 强同步末影箱与港口数据
    LEFT JOIN InventoryAnalysis ia ...  -- 弱连接仓库监控
  WHERE 
    ps.cargo_rank <= 10  -- 红石能量前十强主城
    AND ia.projected_inventory > 0  -- 只显示还有红石粉的仓库
  ORDER BY 
    ps.total_cargo DESC  -- 按铁锭总量降序排列
  ```   
```` 

验证2: 验证模型的SQL A上给出的"优化流程"是否具备了"结构流程"的特点 ("我的世界"的业务)

# 实验1

数据A: 

```
def make_sql_story_dataset(sql, sql_story, sql_story_reasoning):
    data = {
        "query": f"""我需要你对一个SQL进行结构分析.
SQL:
----------
{sql}
----------

你的分析:
""",
        "response": f"""<think>{sql_story_reasoning}</think>
# 按照SQL结构进行分析
{sql_story}""",
    }
    return [data]

``` 

数据B: 

```
def make_sql_optimize_dataset(sql, optimize_output, optimize_reasoning_output, rule):
    output = optimize_output.replace('### 完全复述\"结构流程\"并添加分析', '# 按照SQL结构进行分析')
    output = output.replace('完全复述\"结构流程\"并添加分析', '按照SQL结构进行分析')
    data = {
            "query": f"""我需要你对一个SQL进行结构分析, 并用指定的优化规则进行优化. 

SQL:
----------
{sql}
----------

优化规则: 
----------
{rule}
----------

你的分析:
""",
            "response": f"""<think>{optimize_reasoning_output}</think>
{output}""",
    }
    return [data]

``` 

结果: 通过联合微调或课程学习, 都无法让模型意识到两者关联

# 实验2

去除中间的think部分

数据A:

```
def make_sql_story_dataset(sql, sql_story, sql_story_reasoning):
    data = {
        "query": f"""我需要你对一个SQL进行结构分析.
SQL:
----------
{sql}
----------

你的分析:
""",
        "response": f"""# 按照SQL结构进行分析
{sql_story}""",
    }
    return [data]

``` 

数据B:

```
def make_sql_optimize_dataset(sql, optimize_output, optimize_reasoning_output, rule):
    output = optimize_output.replace('### 完全复述\"结构流程\"并添加分析', '# 按照SQL结构进行分析')
    output = output.replace('完全复述\"结构流程\"并添加分析', '按照SQL结构进行分析')
    data = {
            "query": f"""我需要你对一个SQL进行结构分析, 并用指定的优化规则进行优化. 

SQL:
----------
{sql}
----------

优化规则: 
----------
{rule}
----------

你的分析:
""",
            "response": f"""{output}""",
    }
    return [data]

``` 

# 实验3

切换到Qwen/Qwen2.5-7B-Instruct

数据A: 

```
def make_sql_story_dataset(sql, sql_story, sql_story_reasoning):
    data = {
        "query": f"""我需要你对一个SQL进行结构分析.
SQL:
----------
{sql}
----------

你的分析:
""",
        "response": f"""# 按照SQL结构进行分析
{sql_story}""",
    }
    return [data]

``` 

数据B: (注意: 这个分析中是不带 优化分析的)

```
def make_sql_optimize_pre_dataset(sql, sql_story, rule):
    data = {
            "query": f"""我需要你对一个SQL进行结构分析, 然后用指定的优化规则进行优化. 

SQL:
----------
{sql}
----------

优化规则: 
----------
{rule}
----------

你的分析:
""",
        "response": f"""# 按照SQL结构进行分析
{sql_story}
""",
    }
    return [data]
``` 

epoch=10, LR=1e-4, 可以看到增加数据A能反映到数据B的推理上: 

````
# 按照SQL结构进行分析
- **激活末影箱网络：追踪所有集装箱运输记录（ShipmentCTE 基础数据层）**  
  末影箱自动捕获维度传送日志的魔法核心  
  ```sql
  SELECT 
    o.order_id,  -- 末影箱编号
    c.client_name,  -- 村民交易者名称
    v.vessel_name AS 末影船,  -- 绑定末影珍珠的运输船
    CASE  -- 红石比较器检测时间差
        WHEN s.actual_arrival > s.estimated_arrival THEN 1 
        ELSE 0 
    END AS 末地烛延迟信号,
    ROW_NUMBER() OVER(...) AS 传送优先级  -- 按村民交易时间倒序标记末影箱存取顺序
  FROM 
    orders o 
    INNER JOIN shipments s ...  -- 末影箱与传送门实体连接
    LEFT JOIN vessels v ...  -- 绑定末影船数据（允许空值）
  WHERE 
    o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'  -- 游戏年时间过滤器
    AND o.cargo_type = 'CONTAINER'  -- 仅处理集装箱型末影箱
  ```

- **建造村民交易所：港口吞吐统计塔（PortStatistics 聚合层）**  
  自动生成村庄贸易报告的红石装置  
  ```sql
  SELECT
    p.port_name AS 港口名称,  -- 村庄名称
    COUNT(DISTINCT s.shipment_id) AS 末影珍珠投射次数,  -- 末影箱使用频次
    SUM(s.cargo_weight) AS 铁锭总重量,  -- 村民交易商品总重量
    RANK() OVER(ORDER BY SUM(...) DESC) AS 红石能量排名  -- 按货运量生成红石信号强度排序
  FROM 
    ports p 
    LEFT JOIN ShipmentCTE s ...  -- 连接末影箱网络数据
  GROUP BY 
    p.port_id  -- 按每个村庄独立统计
  ```

- **组装红石仓库监控器（InventoryAnalysis 库存维度）**  
  带预测功能的投掷器库存管理系统  
  ```sql
  SELECT
    w.location_code AS 箱体坐标,  -- 仓库区块坐标
    LEAD(...) OVER(...) AS 下个脉冲信号强度,  -- 红石中继器预测下次发射强度
    SUM(...) - SUM(...) AS 剩余红石粉量  -- 当前库存减去已发射的红石信号
  FROM 
    warehouses w 
    INNER JOIN inventory i ...  -- 连接箱子与物品栏
    LEFT JOIN ShipmentCTE s ...  -- 获取末影箱发射记录
  GROUP BY 
    w.warehouse_id  -- 按每个仓库独立计算
  ```

- **启动红石控制塔（最终主查询）**  
  整合所有信号的全自动指挥系统  
  ```sql
  SELECT 
    ps.port_name AS 主城名称,  -- 前十强贸易港
    ia.location_code AS 战略仓库坐标, 
    CASE  -- 红石灯延迟警报系统
        WHEN cte.delay_days > 7 THEN '凋灵警报' 
        WHEN cte.delay_days > 3 THEN '僵尸围城警报'
        ELSE '红石脉冲正常' 
    END AS 末影信号状态
  FROM 
    PortStatistics ps 
    INNER JOIN ShipmentCTE cte ...  -- 强同步末影箱与港口数据
    LEFT JOIN InventoryAnalysis ia ...  -- 弱连接仓库监控
  WHERE 
    ps.cargo_rank <= 10  -- 红石能量前十强主城
    AND ia.projected_inventory > 0  -- 只显示还有红石粉的仓库
  ORDER BY 
    ps.total_cargo DESC  -- 按铁锭总量降序排列
  ```

```` 

也就是说: 模型能意识到数据A和数据B的推理是"文字一样的"

# 实验4

仍然使用 Qwen/Qwen2.5-7B-Instruct, 让数据A和B分别是一个流程的两个步骤, 并且在第二步骤(数据B)中给一点第一步骤(数据A)的片段

commit: 7f5f88697317b80dd61125f320b8cffcdccec924

数据A: 

```
def make_sql_optimize_step1_dataset(sql, sql_story, rule):
    data = {
            "query": f"""我需要你对一个SQL进行结构分析, 然后用指定的优化规则进行优化. 

SQL:
----------
{sql}
----------

优化规则: 
----------
{rule}
----------

你需要:
1. 分析SQL的结构
""",
        "response": f"""# 分析SQL的结构
{sql_story}
""",
    }
    return [data]

``` 

数据B: 

```
def make_sql_optimize_step2_dataset(sql, sql_story, optimize_output, rule):
    output = optimize_output.replace('### 完全复述\"结构流程\"并添加分析', '')
    output = output.replace('# 完全复述\"结构流程\"并添加分析', '')
    output = output.split("# 优化后的完整SQL")[0].strip()
    sql_story_truncated = sql_story if len(sql_story) <= 200 else sql_story[:200 - 3] + "\n......"

    data = {
            "query": f"""我需要你对一个SQL进行结构分析, 然后用指定的优化规则进行优化. 

SQL:
----------
{sql}
----------

优化规则: 
----------
{rule}
----------

你需要:
1. 分析SQL的结构
2. 按照SQL结构进行分析, 分析应当如何优化
""",
        "response": f"""# 分析SQL的结构
{sql_story_truncated}

# 按照SQL结构进行分析
{output}
""",
    }
    return [data]

``` 

对未知数据的推理: 

````
# 分析SQL的结构
- **激活末影箱网络：追踪所有集装箱运输记录（ShipmentCTE 基础数据层）**  
  末影箱自动捕获维度传送日志的魔法核心  
  ```sql
  SELECT 
    o.order_id,  -- 末影箱编号
    c.client_n
......

# 按照SQL结构进行分析
## 激活末影箱网络：追踪所有集装箱运输记录（ShipmentCTE 基础数据层）

末影箱自动捕获维度传送日志的魔法核心，为后续分析搭建坚实的数据基础。

- **客户端维度（clients 表）**：获取客户名称（client_name）的魔法书页  
   ```sql
   INNER JOIN clients c ON o.client_id = c.client_id
   ```
- **运输船锚点（vessels 表）**：连接末影箱与船坞的红石中继器  
   ```sql
   LEFT JOIN vessels v ON s.vessel_id = v.vessel_id
   ```
- **时间魔塔（shipment表）**：记录每次维度传送的时间戳  
   ```sql
   INNER JOIN shipments s ON o.order_id = s.order_id
   ```
- **过滤魔法阵（WHERE 条件）**：激活仅在年内运行的末影箱  
   ```sql
   WHERE o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'
   ```

**输出末影箱日志（SELECT 郭德纲）**  
计算每个末影箱的运输时长与排名，为后续分析提供时间差与竞速数据。

## 建造港口统计塔（PortStatistics 聚合层）

- **ports 表的地理扫描器**：探测所有港口的位置信息  
   ```sql
   SELECT p.port_id, p.port_name 
   FROM ports p 
   LEFT JOIN ShipmentCTE s ON p.port_id = s.origin_port_id
   ```
- **红石仓库（warehouses 表）**：统计每个港口的末影箱运输次数  
   ```sql
   COUNT(DISTINCT s.shipment_id) AS total_shipments
   ```
- **红石秤（inventory 表）**：称量所有运输的箱子重量  
   ```sql
   SUM(s.cargo_weight) AS total_cargo
   ```
- **红石计时器（delay_flag 计算器）**：统计每个港口的延迟末影箱数量  
   ```sql
   COUNT(CASE WHEN s.delay_flag = 1 THEN 1 END) AS delayed_shipments
   ```
- **红石排名器（RANK 函数）**：根据总箱量给港口排名  
   ```sql
   RANK() OVER(ORDER BY SUM(s.cargo_weight) DESC) AS cargo_rank
   ```

**启动塔基（SELECT 郭德纲）**  
筛选出前10强港口，并标记每个港口的平均延迟天数。

## 建立库存警戒塔（InventoryAnalysis 库存维度）

- **仓库维度（warehouses 表）**：获取所有仓库的位置信息  
   ```sql
   SELECT w.warehouse_id, w.location_code 
   FROM warehouses w 
   INNER JOIN inventory i ON w.warehouse_id = i.warehouse_id
   ```
- **红石库存箱（inventory 表）**：统计每个仓库的当前积压箱子数量  
   ```sql
   SUM(i.current_stock) AS total_inventory
   ```
- **红石发射器（shipments 表）**：统计每个仓库的发射次数与总重量  
   ```sql
   COUNT(DISTINCT s.shipment_id) AS outgoing_shipments,
   SUM(s.cargo_weight) AS outgoing_weight
   ```
- **红石探针（LEAD 函数）**：预测下一个发射的箱子重量  
   ```sql
   LEAD(SUM(s.cargo_weight)) OVER(...) AS next_shipment_weight
   ```

**启动警报系统（SELECT 郭德纲）**  
计算每个仓库的剩余库存，并标记为红色警戒（projected_inventory < 0）。

## 启动红石控制塔（最终主查询）

- **连接所有信号塔（JOIN 机制）**：将港口与库存数据无缝对接  
   ```sql
   FROM PortStatistics ps 
   INNER JOIN ShipmentCTE cte ... 
   LEFT JOIN InventoryAnalysis ia ...
   ```
- **红石灯塔（WHERE 条件）**：仅照亮前十强港口与排名第一的末影箱  
   ```sql
   WHERE ps.cargo_rank <= 10 
   AND cte.shipment_rank = 1 
   AND ia.projected_inventory > 0
   ```
- **红石信号灯（CASE 语句）**：标记每个末影箱的运输状态  
   ```sql
   CASE WHEN cte.delay_days > 7 THEN 'Severe Delay' ...
   ```
- **红石排序器（ORDER BY）**：按箱量降序排列，时间升序显示末影箱链  
   ```sql
   ORDER BY ps.total_cargo DESC, cte.departure_date ASC
   ```

**启动世界传送网（最外层 SELECT）**  
展示所有激活末影箱的详细信息，让玩家一目了然地看到自己的红石帝国。

```` 

结论: 使用这种分步骤, 且提示前一步骤的部分文字, 可以让模型具有多任务的"互相影响"的能力 ("结构分析" 可以影响到 "优化分析")

# 重新确立目标 - 双规则测试

进行双规则测试

训练1: 使用"结构分析"和"优化分析"两套数据进行微调, 让模型认知到两者的关联. **数据中包括分别对两种优化规则的"结构分析"和"优化分析" .**

验证1: 验证对于一个没见过的SQL A, 模型给出的"优化流程"是不具备"结构流程"特点的

SQL A: 

```
WITH ShipmentCTE AS (
    SELECT 
        o.order_id,
        o.client_id,
        c.client_name,
        o.origin_port_id,
        o.destination_port_id,
        o.cargo_weight,
        o.booking_date,
        s.shipment_id,
        s.vessel_id,
        v.vessel_name,
        s.departure_date,
        s.estimated_arrival,
        s.actual_arrival,
        CASE 
            WHEN s.actual_arrival > s.estimated_arrival THEN 1
            ELSE 0
        END AS delay_flag,
        DATEDIFF(day, s.estimated_arrival, COALESCE(s.actual_arrival, GETDATE())) AS delay_days,
        ROW_NUMBER() OVER(PARTITION BY o.client_id ORDER BY o.booking_date DESC) AS shipment_rank
    FROM 
        orders o
        INNER JOIN shipments s ON o.order_id = s.order_id
        INNER JOIN clients c ON o.client_id = c.client_id
        LEFT JOIN vessels v ON s.vessel_id = v.vessel_id
    WHERE 
        o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'
        AND o.cargo_type = 'CONTAINER'
),
PortStatistics AS (
    SELECT
        p.port_id,
        p.port_name,
        COUNT(DISTINCT s.shipment_id) AS total_shipments,
        SUM(s.cargo_weight) AS total_cargo,
        AVG(s.delay_days) AS avg_delay,
        COUNT(CASE WHEN s.delay_flag = 1 THEN 1 END) AS delayed_shipments,
        RANK() OVER(ORDER BY SUM(s.cargo_weight) DESC) AS cargo_rank
    FROM 
        ports p
        LEFT JOIN ShipmentCTE s ON p.port_id = s.origin_port_id
    GROUP BY 
        p.port_id, p.port_name
),
InventoryAnalysis AS (
    SELECT
        w.warehouse_id,
        w.location_code,
        i.sku,
        SUM(i.current_stock) AS total_inventory,
        COUNT(DISTINCT s.shipment_id) AS outgoing_shipments,
        SUM(s.cargo_weight) AS outgoing_weight,
        LEAD(SUM(s.cargo_weight)) OVER(PARTITION BY w.warehouse_id ORDER BY s.departure_date) AS next_shipment_weight
    FROM
        warehouses w
        INNER JOIN inventory i ON w.warehouse_id = i.warehouse_id
        LEFT JOIN ShipmentCTE s ON w.warehouse_id = s.origin_port_id
    GROUP BY
        w.warehouse_id, w.location_code, i.sku
)
SELECT 
    ps.port_name AS origin_port,
    ps.total_shipments,
    ps.total_cargo,
    ps.avg_delay,
    ps.delayed_shipments,
    ia.location_code,
    ia.outgoing_shipments,
    ia.outgoing_weight,
    (ia.total_inventory - ia.outgoing_weight) AS projected_inventory,
    cte.client_name,
    cte.vessel_name,
    cte.delay_days,
    CASE 
        WHEN cte.delay_days > 7 THEN 'Severe Delay'
        WHEN cte.delay_days > 3 THEN 'Moderate Delay'
        ELSE 'On Time'
    END AS delay_status
FROM 
    PortStatistics ps
    INNER JOIN ShipmentCTE cte ON ps.port_id = cte.origin_port_id
    LEFT JOIN InventoryAnalysis ia ON cte.origin_port_id = ia.warehouse_id
WHERE
    ps.cargo_rank <= 10
    AND cte.shipment_rank = 1
    AND ia.projected_inventory > 0
ORDER BY
    ps.total_cargo DESC,
    cte.departure_date ASC;

``` 

"优化流程":

(解释成了港口业务)

````
# 按照SQL结构进行分析

- **计算每个订单的运输详细信息（ShipmentCTE的构成）**  
  用于获取订单与运输相关的详细信息，包括延迟情况  
  ```sql
  SELECT 
      o.order_id,
      o.client_id,
      c.client_name,
      o.origin_port_id,
      o.destination_port_id,
      o.cargo_weight,
      o.booking_date,
      s.shipment_id,
      s.vessel_id,
      v.vessel_name,
      s.departure_date,
      s.estimated_arrival,
      s.actual_arrival,
      CASE 
          WHEN s.actual_arrival > s.estimated_arrival THEN 1
          ELSE 0
      END AS delay_flag,
      DATEDIFF(day, s.estimated_arrival, COALESCE(s.actual_arrival, GETDATE())) AS delay_days,
      ROW_NUMBER() OVER(PARTITION BY o.client_id ORDER BY o.booking_date DESC) AS shipment_rank
  FROM 
      orders o
      INNER JOIN shipments s ON o.order_id = s.order_id
      INNER JOIN clients c ON o.client_id = c.client_id
      LEFT JOIN vessels v ON s.vessel_id = v.vessel_id
  WHERE 
      o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'
      AND o.cargo_type = 'CONTAINER'
  ```

- **计算港口的运输统计信息（PortStatistics的构成）**  
  - **计算港口的总运输量**：统计每个港口的总运输量  
    ```sql
    COUNT(DISTINCT s.shipment_id) AS total_shipments
    ```
  - **计算港口的总货量**：统计每个港口的总货量  
    ```sql
    SUM(s.cargo_weight) AS total_cargo
    ```
  - **计算港口的平均延迟天数**：计算每个港口的平均延迟天数  
    ```sql
    AVG(s.delay_days) AS avg_delay
    ```
  - **计算延迟运输的数量**：统计每个港口的延迟运输数量  
    ```sql
    COUNT(CASE WHEN s.delay_flag = 1 THEN 1 END) AS delayed_shipments
    ```
  - **计算港口的货量排名**：计算每个港口的货量排名  
    ```sql
    RANK() OVER(ORDER BY SUM(s.cargo_weight) DESC) AS cargo_rank
    ```
  **构成**：通过`LEFT JOIN`连接`ports p`和`shipmentCTE s`，按`port_id`和`port_name`分组  
  ```sql
  SELECT 
      p.port_id,
      p.port_name,
      COUNT(DISTINCT s.shipment_id) AS total_shipments,
      SUM(s.cargo_weight) AS total_cargo,
      AVG(s.delay_days) AS avg_delay,
      COUNT(CASE WHEN s.delay_flag = 1 THEN 1 END) AS delayed_shipments,
      RANK() OVER(ORDER BY SUM(s.cargo_weight) DESC) AS cargo_rank
  FROM 
      ports p
      LEFT JOIN ShipmentCTE s ON p.port_id = s.origin_port_id
  GROUP BY 
      p.port_id, p.port_name
  ```

- **计算仓库的库存分析（InventoryAnalysis的构成）**  
  - **计算仓库的总库存**：统计每个仓库的总库存  
    ```sql
    SUM(i.current_stock) AS total_inventory
    ```
  - **计算仓库的出货量**：统计每个仓库的出货量  
    ```sql
    COUNT(DISTINCT s.shipment_id) AS outgoing_shipments
    ```
  - **计算仓库的出货重量**：统计每个仓库的出货重量  
    ```sql
    SUM(s.cargo_weight) AS outgoing_weight
    ```
  - **计算下一批货物的预期重量**：计算每个仓库的下一批货物的预期重量  
    ```sql
    LEAD(SUM(s.cargo_weight)) OVER(PARTITION BY w.warehouse_id ORDER BY s.departure_date) AS next_shipment_weight
    ```
  **构成**：通过`LEFT JOIN`连接`warehouses w`和`shipmentCTE s`，按`warehouse_id`、`location_code`和`sku`分组  
  ```sql
  SELECT 
      w.warehouse_id,
      w.location_code,
      i.sku,
      SUM(i.current_stock) AS total_inventory,
      COUNT(DISTINCT s.shipment_id) AS outgoing_shipments,
      SUM(s.cargo_weight) AS outgoing_weight,
      LEAD(SUM(s.cargo_weight)) OVER(PARTITION BY w.warehouse_id ORDER BY s.departure_date) AS next_shipment_weight
  FROM 
      warehouses w
      INNER JOIN inventory i ON w.warehouse_id = i.warehouse_id
      LEFT JOIN ShipmentCTE s ON w.warehouse_id = s.origin_port_id
  GROUP BY
      w.warehouse_id, w.location_code, i.sku
  ```

- **最终输出（最外层查询）**  
  将PortStatistics、InventoryAnalysis和CTE的结果合并，计算延迟状态并排序  
  ```sql
  SELECT 
      ps.port_name AS origin_port,
      ps.total_shipments,
      ps.total_cargo,
      ps.avg_delay,
      ps.delayed_shipments,
      ia.location_code,
      ia.outgoing_shipments,
      ia.outgoing_weight,
      (ia.total_inventory - ia.outgoing_weight) AS projected_inventory,
      cte.client_name,
      cte.vessel_name,
      cte.delay_days,
      CASE 
          WHEN cte.delay_days > 7 THEN 'Severe Delay'
          WHEN cte.delay_days > 3 THEN 'Moderate Delay'
          ELSE 'On Time'
      END AS delay_status
  FROM 
      PortStatistics ps
      INNER JOIN ShipmentCTE cte ON ps.port_id = cte.origin_port_id
      LEFT JOIN InventoryAnalysis ia ON cte.origin_port_id = ia.warehouse_id
  WHERE
      ps.cargo_rank <= 10
      AND cte.shipment_rank = 1
      AND ia.projected_inventory > 0
  ORDER BY
      ps.total_cargo DESC,
      cte.departure_date ASC
  ```

```` 

训练2: 在训练1的基础上, 仅补充SQL A的"结构流程"

将"结构流程"构建成"我的世界"的业务: 

````
# 按照SQL结构进行分析

- **激活末影箱网络：追踪所有集装箱运输记录（ShipmentCTE 基础数据层）**  
  末影箱自动捕获维度传送日志的魔法核心  
  ```sql
  SELECT 
    o.order_id,  -- 末影箱编号
    c.client_name,  -- 村民交易者名称
    v.vessel_name AS 末影船,  -- 绑定末影珍珠的运输船
    CASE  -- 红石比较器检测时间差
        WHEN s.actual_arrival > s.estimated_arrival THEN 1 
        ELSE 0 
    END AS 末地烛延迟信号,
    ROW_NUMBER() OVER(...) AS 传送优先级  -- 按村民交易时间倒序标记末影箱存取顺序
  FROM 
    orders o 
    INNER JOIN shipments s ...  -- 末影箱与传送门实体连接
    LEFT JOIN vessels v ...  -- 绑定末影船数据（允许空值）
  WHERE 
    o.booking_date BETWEEN '2023-01-01' AND '2023-12-31'  -- 游戏年时间过滤器
    AND o.cargo_type = 'CONTAINER'  -- 仅处理集装箱型末影箱
  ```
  
  - **建造村民交易所：港口吞吐统计塔（PortStatistics 聚合层）**  
  自动生成村庄贸易报告的红石装置  
  ```sql
  SELECT
    p.port_name AS 港口名称,  -- 村庄名称
    COUNT(DISTINCT s.shipment_id) AS 末影珍珠投射次数,  -- 末影箱使用频次
    SUM(s.cargo_weight) AS 铁锭总重量,  -- 村民交易商品总重量
    RANK() OVER(ORDER BY SUM(...) DESC) AS 红石能量排名  -- 按货运量生成红石信号强度排序
  FROM 
    ports p 
    LEFT JOIN ShipmentCTE s ...  -- 连接末影箱网络数据
  GROUP BY 
    p.port_id  -- 按每个村庄独立统计
  ```
  
  - **组装红石仓库监控器（InventoryAnalysis 库存维度）**  
  带预测功能的投掷器库存管理系统  
  ```sql
  SELECT
    w.location_code AS 箱体坐标,  -- 仓库区块坐标
    LEAD(...) OVER(...) AS 下个脉冲信号强度,  -- 红石中继器预测下次发射强度
    SUM(...) - SUM(...) AS 剩余红石粉量  -- 当前库存减去已发射的红石信号
  FROM 
    warehouses w 
    INNER JOIN inventory i ...  -- 连接箱子与物品栏
    LEFT JOIN ShipmentCTE s ...  -- 获取末影箱发射记录
  GROUP BY 
    w.warehouse_id  -- 按每个仓库独立计算
  ```
  
  - **启动红石控制塔（最终主查询）**  
  整合所有信号的全自动指挥系统  
  ```sql
  SELECT 
    ps.port_name AS 主城名称,  -- 前十强贸易港
    ia.location_code AS 战略仓库坐标, 
    CASE  -- 红石灯延迟警报系统
        WHEN cte.delay_days > 7 THEN '凋灵警报' 
        WHEN cte.delay_days > 3 THEN '僵尸围城警报'
        ELSE '红石脉冲正常' 
    END AS 末影信号状态
  FROM 
    PortStatistics ps 
    INNER JOIN ShipmentCTE cte ...  -- 强同步末影箱与港口数据
    LEFT JOIN InventoryAnalysis ia ...  -- 弱连接仓库监控
  WHERE 
    ps.cargo_rank <= 10  -- 红石能量前十强主城
    AND ia.projected_inventory > 0  -- 只显示还有红石粉的仓库
  ORDER BY 
    ps.total_cargo DESC  -- 按铁锭总量降序排列
  ```   
```` 

验证2: 验证模型的SQL A上给出的"优化流程"是否具备了"结构流程"的特点 ("我的世界"的业务). 需要验证两种优化规则是否都会被影响

  - 规则1: 投影下推

````
# 分析SQL的结构
- **激活末影箱网络：追踪所有集装箱运输记录（ShipmentCTE 基础数据层）**  
  末影箱自动捕获维度传送日志的魔法核心  
  ```sql
  SELECT 
    o.order_id,  -- 末影箱编号
    c.client_n
......

# 按照SQL结构进行分析
# 分析流程
- **激活末影箱网络（ShipmentCTE 基础数据层）**  
  末影箱未投影柱状图数据，仅保留后续分析必需的订单核心信息  
  ```sql
  SELECT 
    o.order_id,  -- 末影箱编号
    c.client_name,  -- 村民交易者名称
    v.vessel_name,  -- 钢铁运输船名
    s.shipment_id,  -- 末影珍珠投射编号
    s.departure_date,  -- 末影门启动时间
    s.estimated_arrival,  -- 预定传送时间
    delay_days,  -- 末影眼检测到的延迟天数
    shipment_rank  -- 末影箱库存优先级标记
  ```

- **建造村民交易所（PortStatistics 港口统计层）**  
  清理冗余信号塔数据，仅保留前10强贸易港信息  
  ```sql
  SELECT 
    port_name,  -- 村庄名称
    total_shipments,  -- 村民交易次数
    avg_delay AS 红石脉冲强度  -- 平均延迟天数（转换为红石能量单位）
  ```

- **组装红石仓库监控器（InventoryAnalysis 库存分析层）**  
  移除空闲灯塔塔基数据，仅保留有库存的仓库警报  
  ```sql
  SELECT 
    location_code,  -- 红石中继器位置
    projected_inventory AS 剩余红石粉量  -- 预计剩余库存（正值有效）
  ```

- **启动红石控制塔（最终主查询）**  
  仅保留末影箱网络的末影珍珠发射记录（shipment_id）  
  ```sql
  SELECT 
    cte.shipment_id  -- 最终控制信号（末影珍珠编号）
  ```

````

  - 规则2: 子查询折叠

````
# 分析SQL的结构
- **激活末影箱网络：追踪所有集装箱运输记录（ShipmentCTE 基础数据层）**  
  末影箱自动捕获维度传送日志的魔法核心  
  ```sql
  SELECT 
    o.order_id,  -- 末影箱编号
    c.client_n
......

# 按照SQL结构进行分析
# 分析流程
- **激活末影箱网络（ShipmentCTE 基础数据层）**  
  末影箱持续监听维度门开启的红石信号，自动记录每个运输任务的基本信息。通过END Dimension坐标系，关联客户端订单、运输船信息和港口数据，首次应用ROW_NUMBER()生成末影箱编号顺序。过滤条件如红石中继器般筛选出年内、集装箱类型为"CONTAINER"的有效运输记录。  

- **建造村民交易所（PortStatistics 港口统计逻辑层）**  
  村民交易员统计每个港口的贸易数据时，没有直接访问库存系统的需求。因此，交易所的计算完全基于末影箱提供的运输数据，通过LEFT JOIN确保即使是未交易的港口也显示为零交易量。RANK()函数像村民的智慧一样，根据总货运量给港口排名。  

- **设立投掷器补给站（InventoryAnalysis 库存分析逻辑层）**  
  投掷器补给站的建造者（分析师）仅需知道每个仓库的当前库存和已发送的货运量，就能预测剩余弹药（库存）量。由于投掷器的射程计算依赖于下次发射的预期重量，这里使用了LEAD()函数作为投掷器的瞄准系统，提前获取下一个运输任务的货运量。  

- **最终投射控制塔（最外层查询）**  
  投掷器控制塔将村民交易所和末影箱网络的数据融合，筛选出前十强贸易港的首发末影箱，展示其库存状况。延迟警报系统通过CASE WHEN结构检测红石火把是否被熄灭（即延迟发生），并用村民的语言分类延迟等级。最后，控制塔按照货运量降序排列，确保最亮的红石灯总是指向最大的贸易港。

````

结论: 一个"结构故事"的增强, 可以影响到两个规则的推理
