---
title: 20250710 - 弘讯数据分析中发现的问题
confluence_page_id: 4161606
created_at: 2025-07-10T14:52:08+00:00
updated_at: 2025-07-12T14:53:53+00:00
---

1. lotslog 和 workorder 的起止时间对不上
  2. online_craft, 对于同一个机器, 同一个craft_sn, 仍然有多个记录?
     1. 其对历史数据分析好像无用
  3. additives（添加剂表）为空
  4. material_add_record 只有两条三月的记录, material_return_record 没有记录, material 只有一条数据
  5. maintain_*中, 只有一台机器的记录, 或没有记录
  6. machine_fault 没有记录
  7. shotcounthistory 没有记录
  8. sync_standdb_log 只有一个记录
  9. tmdatatypes 没有记录
  10. 忽略user*
  11. yield_saved 没有记录
  12. spc_* 没有记录
  13. quality_and_temperature_config ??
  14. qm_* 没有记录
  15. production_inferior 产品次品, 没有记录
  16. product_subitem, product_mold_relation, product_blocked_reason, 没有记录
  17. produce_craft.product_id 跟 上下游的 product_id不一致
  18. produce_class 只有一条记录
  19. processes (作业): 看上去是定义了界面层级和按钮?
  20. processes_actions (角色): 是一些操作按钮? 
  21. parammodifyhistory 无数据
  22. operate_status, operate_status_record: 看上去是机器的操作状态? 有数据, 但状态值常量没有说明
  23. opcua_data, opcua_category: 看上去是"数据点位字典", 配置: 数据在PLC机器中的地址
  24. monitorhistory, 无数据
  25. mold_maintain_unqualified, mold_maintain_spotcheck: 无数据
  26. ipqc_inspection_main, IPQC 质检, 只有几条数据?
  27. inetcraftplater, 只有一条数据
  28. hmi54dataid2stddataid/hmidatametas/hmisystemdatabase/hmisystemdmetarinetdmeta/hmiversiondefine/hmiversionmetarinetmeta/hmiversionmetarinetmetainherit :
     1. 注塑机上的人机交互pad的配置元数据
  29. hive_info, 只有一条数据
  30. factory_area, 先忽略
  31. energe*, 没有数据
  32. deviceshowdatas 中, device_type=doubleColor的作用?
  33. blocked_*, 无数据
  34. alarmhistory, 无数据. alarm_infor是错误代码
  35. additive, 无数据
  36. report_statistics_hour 和 report_statistics_day 能耗不一致
