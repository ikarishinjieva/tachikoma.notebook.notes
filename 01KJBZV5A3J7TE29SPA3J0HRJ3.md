---
title: 20250712 - 弘讯数据分析 - 根据时间线, 整理所有跟生产过程有关的数据
confluence_page_id: 4161626
created_at: 2025-07-11T17:24:35+00:00
updated_at: 2025-07-14T08:02:33+00:00
---

# 背景

弘讯给出了 注塑机系统的数据库, 经过整理: 

  - <https://github.com/ikarishinjieva/ai-fast-demos/tree/main/hongxun_data_viewer> , 用AI做了一个数据查看器, 将主要数据表进行了关联关系整理
  - [20250710 - 弘讯数据分析中发现的问题] , 将数据发现的问题整理在这里

当前状态: 得到了数据表的关系, 需要按时间线, 整理跟生产过程相关的数据, 包括: 

  - 实时数据: devicedatanew
  - 批次: lotslog
  - 每模的工艺历史: per_shotcount_craft
  - 生产日志细节: productionlog
  - 停机原因: mach_shutdown_reason_record
  - 工单: workorder
  - 订单状态变更: production_order_status
  - 能量报告: report_statistics_day
  - 点检: spot_check_craft

# 界面样例

![image2025-7-13 23:20:48.png](/assets/01KJBZV5A3J7TE29SPA3J0HRJ3/image2025-7-13%2023%3A20%3A48.png)

# 校验工艺变更记录

校验工艺变更记录和点检之间的关系, 使用脚本: verify_craft.js : 

```
node verify_craft.js "-WmEvGqWQmasZkQJGAKVgg" "2025-07-04T00:00:00Z" "2025-07-07T00:00:00Z"
``` 

结果: 两者是一致的

[附件: 2.txt] 

通过"点检" 确定全量, "每模工艺历史"确定增量, 就可以追踪工艺变更的精确时间和历史

# 发现的问题

  1. 点检的报告, 比实时数据 (devicedatanew), 会多出一些参数: 
     1. ![image2025-7-13 23:20:14.png](/assets/01KJBZV5A3J7TE29SPA3J0HRJ3/image2025-7-13%2023%3A20%3A14.png)
  2. 实时数据的频次很低
     1. ![image2025-7-13 23:20:48.png](/assets/01KJBZV5A3J7TE29SPA3J0HRJ3/image2025-7-13%2023%3A20%3A48.png)
  3. 批次和生产日志的关系:
     1. 批次的actual_output (实际产出) 和 批次中production_log的total_qty总和 对比
        1. 批次的实际产出 = 生产日志数量总和 : 44669 个
        2. 批次的实际产出 < 生产日志数量总和 : 47 个
        3. 批次的实际产出 > 生产日志数量总和 : 1942 个
  4. 批次和工单的关系, 对不上

3\.
