---
title: 20231027 - 南航 ActionDB 3.x和4.x执行计划不一样
confluence_page_id: 2589300
created_at: 2023-10-27T15:34:50+00:00
updated_at: 2023-10-30T03:29:31+00:00
---

# 

# 表定义

```
| act_transaction | CREATE TABLE `act_transaction` (
  `txn_id` decimal(20,0) NOT NULL COMMENT '交易id',
  `evt_id` decimal(20,0) DEFAULT NULL COMMENT '事件id',
  `txn_typ` decimal(10,0) DEFAULT NULL COMMENT '交易类型',
  `txn_cls` varchar(3) DEFAULT NULL COMMENT '交易分类',
  `points_amt` decimal(19,3) DEFAULT NULL COMMENT '积分总额',
  `points_typ` varchar(3) DEFAULT NULL COMMENT '积分类型',
  `effect_tm` timestamp NULL DEFAULT NULL COMMENT '生效时间',
  `ins_tm` timestamp NULL DEFAULT NULL COMMENT '插入时间',
  `ins_dt` date DEFAULT NULL COMMENT '插入日期',
  `tx_date` date DEFAULT NULL COMMENT 'tx_date',
  `src_system` char(3) DEFAULT NULL COMMENT 'src_system',
  `etl_time` timestamp NULL DEFAULT NULL COMMENT 'etl_time',
  KEY `index_Evt_ID` (`evt_id`) BLOCK_SIZE 16384 LOCAL
) DEFAULT CHARSET = utf8mb4 ROW_FORMAT = DYNAMIC COMPRESSION = 'zstd_1.3.8' REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
 partition by range columns(Ins_Dt)
(partition M190001 values less than ('1900-02-01'),
partition M201201 values less than ('2012-02-01'),
partition M201202 values less than ('2012-03-01'),
partition M201203 values less than ('2012-04-01'),
partition M201204 values less than ('2012-05-01'),
partition M201205 values less than ('2012-06-01'),
partition M201206 values less than ('2012-07-01'),
partition M201207 values less than ('2012-08-01'),
partition M201208 values less than ('2012-09-01'),
partition M201209 values less than ('2012-10-01'),
partition M201210 values less than ('2012-11-01'),
partition M201211 values less than ('2012-12-01'),
partition M201212 values less than ('2013-01-01'),
partition M201301 values less than ('2013-02-01'),
partition M201302 values less than ('2013-03-01'),
partition M201303 values less than ('2013-04-01'),
partition M201304 values less than ('2013-05-01'),
partition M201305 values less than ('2013-06-01'),
partition M201306 values less than ('2013-07-01'),
partition M201307 values less than ('2013-08-01'),
partition M201308 values less than ('2013-09-01'),
partition M201309 values less than ('2013-10-01'),
partition M201310 values less than ('2013-11-01'),
partition M201311 values less than ('2013-12-01'),
partition M201312 values less than ('2014-01-01'),
partition M201401 values less than ('2014-02-01'),
partition M201402 values less than ('2014-03-01'),
partition M201403 values less than ('2014-04-01'),
partition M201404 values less than ('2014-05-01'),
partition M201405 values less than ('2014-06-01'),
partition M201406 values less than ('2014-07-01'),
partition M201407 values less than ('2014-08-01'),
partition M201408 values less than ('2014-09-01'),
partition M201409 values less than ('2014-10-01'),
partition M201410 values less than ('2014-11-01'),
partition M201411 values less than ('2014-12-01'),
partition M201412 values less than ('2015-01-01'),
partition M201501 values less than ('2015-02-01'),
partition M201502 values less than ('2015-03-01'),
partition M201503 values less than ('2015-04-01'),
partition M201504 values less than ('2015-05-01'),
partition M201505 values less than ('2015-06-01'),
partition M201506 values less than ('2015-07-01'),
partition M201507 values less than ('2015-08-01'),
partition M201508 values less than ('2015-09-01'),
partition M201509 values less than ('2015-10-01'),
partition M201510 values less than ('2015-11-01'),
partition M201511 values less than ('2015-12-01'),
partition M201512 values less than ('2016-01-01'),
partition M201601 values less than ('2016-02-01'),
partition M201602 values less than ('2016-03-01'),
partition M201603 values less than ('2016-04-01'),
partition M201604 values less than ('2016-05-01'),
partition M201605 values less than ('2016-06-01'),
partition M201606 values less than ('2016-07-01'),
partition M201607 values less than ('2016-08-01'),
partition M201608 values less than ('2016-09-01'),
partition M201609 values less than ('2016-10-01'),
partition M201610 values less than ('2016-11-01'),
partition M201611 values less than ('2016-12-01'),
partition M201612 values less than ('2017-01-01'),
partition M201701 values less than ('2017-02-01'),
partition M201702 values less than ('2017-03-01'),
partition M201703 values less than ('2017-04-01'),
partition M201704 values less than ('2017-05-01'),
partition M201705 values less than ('2017-06-01'),
partition M201706 values less than ('2017-07-01'),
partition M201707 values less than ('2017-08-01'),
partition M201708 values less than ('2017-09-01'),
partition M201709 values less than ('2017-10-01'),
partition M201710 values less than ('2017-11-01'),
partition M201711 values less than ('2017-12-01'),
partition M201712 values less than ('2018-01-01'),
partition M201801 values less than ('2018-02-01'),
partition M201802 values less than ('2018-03-01'),
partition M201803 values less than ('2018-04-01'),
partition M201804 values less than ('2018-05-01'),
partition M201805 values less than ('2018-06-01'),
partition M201806 values less than ('2018-07-01'),
partition M201807 values less than ('2018-08-01'),
partition M201808 values less than ('2018-09-01'),
partition M201809 values less than ('2018-10-01'),
partition M201810 values less than ('2018-11-01'),
partition M201811 values less than ('2018-12-01'),
partition M201812 values less than ('2019-01-01'),
partition M201901 values less than ('2019-02-01'),
partition M201902 values less than ('2019-03-01'),
partition M201903 values less than ('2019-04-01'),
partition M201904 values less than ('2019-05-01'),
partition M201905 values less than ('2019-06-01'),
partition M201906 values less than ('2019-07-01'),
partition M201907 values less than ('2019-08-01'),
partition M201908 values less than ('2019-09-01'),
partition M201909 values less than ('2019-10-01'),
partition M201910 values less than ('2019-11-01'),
partition M201911 values less than ('2019-12-01'),
partition M201912 values less than ('2020-01-01'),
partition M202001 values less than ('2020-02-01'),
partition M202002 values less than ('2020-03-01'),
partition M202003 values less than ('2020-04-01'),
partition M202004 values less than ('2020-05-01'),
partition M202005 values less than ('2020-06-01'),
partition M202006 values less than ('2020-07-01'),
partition M202007 values less than ('2020-08-01'),
partition M202008 values less than ('2020-09-01'),
partition M202009 values less than ('2020-10-01'),
partition M202010 values less than ('2020-11-01'),
partition M202011 values less than ('2020-12-01'),
partition M202012 values less than ('2021-01-01'),
partition M202101 values less than ('2021-02-01'),
partition M202102 values less than ('2021-03-01'),
partition M202103 values less than ('2021-04-01'),
partition M202104 values less than ('2021-05-01'),
partition M202105 values less than ('2021-06-01'),
partition M202106 values less than ('2021-07-01'),
partition M202107 values less than ('2021-08-01'),
partition M202108 values less than ('2021-09-01'),
partition M202109 values less than ('2021-10-01'),
partition M202110 values less than ('2021-11-01'),
partition M202111 values less than ('2021-12-01'),
partition M202112 values less than ('2022-01-01'),
partition M202201 values less than ('2022-02-01'),
partition M202202 values less than ('2022-03-01'),
partition M202203 values less than ('2022-04-01'),
partition M202204 values less than ('2022-05-01'),
partition M202205 values less than ('2022-06-01'),
partition M202206 values less than ('2022-07-01'),
partition M202207 values less than ('2022-08-01'),
partition M202208 values less than ('2022-09-01'),
partition M202209 values less than ('2022-10-01'),
partition M202210 values less than ('2022-11-01'),
partition M202211 values less than ('2022-12-01'),
partition M202212 values less than ('2023-01-01'),
partition M202301 values less than ('2023-02-01'),
partition M202302 values less than ('2023-03-01'),
partition M202303 values less than ('2023-04-01'),
partition M202304 values less than ('2023-05-01'),
partition M202305 values less than ('2023-06-01'),
partition M202306 values less than ('2023-07-01'),
partition M202307 values less than ('2023-08-01'),
partition M202308 values less than ('2023-09-01'),
partition M202309 values less than ('2023-10-01'),
partition M202310 values less than ('2023-11-01'),
partition M202311 values less than ('2023-12-01'),
partition M202312 values less than ('2024-01-01'),
partition M202401 values less than ('2024-02-01'),
partition M202402 values less than ('2024-03-01'),
partition M202403 values less than ('2024-04-01'),
partition M202404 values less than ('2024-05-01'),
partition M202405 values less than ('2024-06-01'),
partition M202406 values less than ('2024-07-01'),
partition M202407 values less than ('2024-08-01'),
partition M202408 values less than ('2024-09-01'),
partition M202409 values less than ('2024-10-01'),
partition M202410 values less than ('2024-11-01'),
partition M202411 values less than ('2024-12-01'),
partition M202412 values less than ('2025-01-01'),
partition M202501 values less than ('2025-02-01'),
partition M202502 values less than ('2025-03-01'),
partition M202503 values less than ('2025-04-01'),
partition M202504 values less than ('2025-05-01'),
partition M202505 values less than ('2025-06-01'),
partition M202506 values less than ('2025-07-01'),
partition M202507 values less than ('2025-08-01'),
partition M202508 values less than ('2025-09-01'),
partition M202509 values less than ('2025-10-01'),
partition M202510 values less than ('2025-11-01'),
partition M202511 values less than ('2025-12-01'),
partition M202512 values less than ('2026-01-01'),
partition M202601 values less than ('2026-02-01'),
partition M202602 values less than ('2026-03-01'),
partition M202603 values less than ('2026-04-01'),
partition M202604 values less than ('2026-05-01'),
partition M202605 values less than ('2026-06-01'),
partition M202606 values less than ('2026-07-01'),
partition M202607 values less than ('2026-08-01'),
partition M202608 values less than ('2026-09-01'),
partition M202609 values less than ('2026-10-01'),
partition M202610 values less than ('2026-11-01'),
partition M202611 values less than ('2026-12-01'),
partition M202612 values less than ('2027-01-01'),
partition M202701 values less than ('2027-02-01'),
partition M202702 values less than ('2027-03-01'),
partition M202703 values less than ('2027-04-01'),
partition M202704 values less than ('2027-05-01'),
partition M202705 values less than ('2027-06-01'),
partition M202706 values less than ('2027-07-01'),
partition M202707 values less than ('2027-08-01'),
partition M202708 values less than ('2027-09-01'),
partition M202709 values less than ('2027-10-01'),
partition M202710 values less than ('2027-11-01'),
partition M202711 values less than ('2027-12-01'),
partition M202712 values less than ('2028-01-01'),
partition M202801 values less than ('2028-02-01'),
partition M202802 values less than ('2028-03-01'),
partition M202803 values less than ('2028-04-01'),
partition M202804 values less than ('2028-05-01'),
partition M202805 values less than ('2028-06-01'),
partition M202806 values less than ('2028-07-01'),
partition M202807 values less than ('2028-08-01'),
partition M202808 values less than ('2028-09-01'),
partition M202809 values less than ('2028-10-01'),
partition M202810 values less than ('2028-11-01'),
partition M202811 values less than ('2028-12-01'),
partition M202812 values less than ('2029-01-01'),
partition M202901 values less than ('2029-02-01'),
partition M202902 values less than ('2029-03-01'),
partition M202903 values less than ('2029-04-01'),
partition M202904 values less than ('2029-05-01'),
partition M202905 values less than ('2029-06-01'),
partition M202906 values less than ('2029-07-01'),
partition M202907 values less than ('2029-08-01'),
partition M202908 values less than ('2029-09-01'),
partition M202909 values less than ('2029-10-01'),
partition M202910 values less than ('2029-11-01'),
partition M202911 values less than ('2029-12-01'),
partition M202912 values less than ('2030-01-01'),
partition M203001 values less than ('2030-02-01'),
partition M203002 values less than ('2030-03-01'),
partition M203003 values less than ('2030-04-01'),
partition M203004 values less than ('2030-05-01'),
partition M203005 values less than ('2030-06-01'),
partition M203006 values less than ('2030-07-01'),
partition M203007 values less than ('2030-08-01'),
partition M203008 values less than ('2030-09-01'),
partition M203009 values less than ('2030-10-01'),
partition M203010 values less than ('2030-11-01'),
partition M203011 values less than ('2030-12-01'),
partition M203012 values less than ('2031-01-01'),
partition M203101 values less than ('2031-02-01'),
partition M203102 values less than ('2031-03-01'),
partition M203103 values less than ('2031-04-01'),
partition M203104 values less than ('2031-05-01'),
partition M203105 values less than ('2031-06-01'),
partition M203106 values less than ('2031-07-01'),
partition M203107 values less than ('2031-08-01'),
partition M203108 values less than ('2031-09-01'),
partition M203109 values less than ('2031-10-01'),
partition M203110 values less than ('2031-11-01'),
partition M203111 values less than ('2031-12-01'),
partition M203112 values less than ('2032-01-01'),
partition M203201 values less than ('2032-02-01'),
partition M203202 values less than ('2032-03-01'),
partition M203203 values less than ('2032-04-01'),
partition M203204 values less than ('2032-05-01'),
partition M203205 values less than ('2032-06-01'),
partition M203206 values less than ('2032-07-01'),
partition M203207 values less than ('2032-08-01'),
partition M203208 values less than ('2032-09-01'),
partition M203209 values less than ('2032-10-01'),
partition M203210 values less than ('2032-11-01'),
partition M203211 values less than ('2032-12-01'),
partition M203212 values less than ('2033-01-01'),
partition M203301 values less than ('2033-02-01'),
partition M203302 values less than ('2033-03-01'),
partition M203303 values less than ('2033-04-01'),
partition M203304 values less than ('2033-05-01'),
partition M203305 values less than ('2033-06-01'),
partition M203306 values less than ('2033-07-01'),
partition M203307 values less than ('2033-08-01'),
partition M203308 values less than ('2033-09-01'),
partition M203309 values less than ('2033-10-01'),
partition M203310 values less than ('2033-11-01'),
partition M203311 values less than ('2033-12-01'),
partition M203312 values less than ('2034-01-01'),
partition M203401 values less than ('2034-02-01'),
partition M203402 values less than ('2034-03-01'),
partition M203403 values less than ('2034-04-01'),
partition M203404 values less than ('2034-05-01'),
partition M203405 values less than ('2034-06-01'),
partition M203406 values less than ('2034-07-01'),
partition M203407 values less than ('2034-08-01'),
partition M203408 values less than ('2034-09-01'),
partition M203409 values less than ('2034-10-01'),
partition M203410 values less than ('2034-11-01'),
partition M203411 values less than ('2034-12-01'),
partition M203412 values less than ('2035-01-01'),
partition M203501 values less than ('2035-02-01'),
partition M203502 values less than ('2035-03-01'),
partition M203503 values less than ('2035-04-01'),
partition M203504 values less than ('2035-05-01'),
partition M203505 values less than ('2035-06-01'),
partition M203506 values less than ('2035-07-01'),
partition M203507 values less than ('2035-08-01'),
partition M203508 values less than ('2035-09-01'),
partition M203509 values less than ('2035-10-01'),
partition M203510 values less than ('2035-11-01'),
partition M203511 values less than ('2035-12-01'),
partition M203512 values less than ('2036-01-01'),
partition M203601 values less than ('2036-02-01'),
partition M203602 values less than ('2036-03-01'),
partition M203603 values less than ('2036-04-01'),
partition M203604 values less than ('2036-05-01'),
partition M203605 values less than ('2036-06-01'),
partition M203606 values less than ('2036-07-01'),
partition M203607 values less than ('2036-08-01'),
partition M203608 values less than ('2036-09-01'),
partition M203609 values less than ('2036-10-01'),
partition M203610 values less than ('2036-11-01'),
partition M203611 values less than ('2036-12-01'),
partition M203612 values less than ('2037-01-01'),
partition M203701 values less than ('2037-02-01'),
partition M203702 values less than ('2037-03-01'),
partition M203703 values less than ('2037-04-01'),
partition M203704 values less than ('2037-05-01'),
partition M203705 values less than ('2037-06-01'),
partition M203706 values less than ('2037-07-01'),
partition M203707 values less than ('2037-08-01'),
partition M203708 values less than ('2037-09-01'),
partition M203709 values less than ('2037-10-01'),
partition M203710 values less than ('2037-11-01'),
partition M203711 values less than ('2037-12-01'),
partition M203712 values less than ('2038-01-01'),
partition M203801 values less than ('2038-02-01'),
partition M203802 values less than ('2038-03-01'),
partition M203803 values less than ('2038-04-01'),
partition M203804 values less than ('2038-05-01'),
partition M203805 values less than ('2038-06-01'),
partition M203806 values less than ('2038-07-01'),
partition M203807 values less than ('2038-08-01'),
partition M203808 values less than ('2038-09-01'),
partition M203809 values less than ('2038-10-01'),
partition M203810 values less than ('2038-11-01'),
partition M203811 values less than ('2038-12-01'),
partition M203812 values less than ('2039-01-01'),
partition M203901 values less than ('2039-02-01'),
partition M203902 values less than ('2039-03-01'),
partition M203903 values less than ('2039-04-01'),
partition M203904 values less than ('2039-05-01'),
partition M203905 values less than ('2039-06-01'),
partition M203906 values less than ('2039-07-01'),
partition M203907 values less than ('2039-08-01'),
partition M203908 values less than ('2039-09-01'),
partition M203909 values less than ('2039-10-01'),
partition M203910 values less than ('2039-11-01'),
partition M203911 values less than ('2039-12-01'),
partition M203912 values less than ('2040-01-01'),
partition M204001 values less than ('2040-02-01'),
partition M204002 values less than ('2040-03-01'),
partition M204003 values less than ('2040-04-01'),
partition M204004 values less than ('2040-05-01'),
partition M204005 values less than ('2040-06-01'),
partition M204006 values less than ('2040-07-01'),
partition M204007 values less than ('2040-08-01'),
partition M204008 values less than ('2040-09-01'),
partition M204009 values less than ('2040-10-01'),
partition M204010 values less than ('2040-11-01'),
partition M204011 values less than ('2040-12-01'),
partition MMAX values less than (MAXVALUE)) |

| act_event | CREATE TABLE `act_event` (
  `evt_id` decimal(20,0) DEFAULT NULL COMMENT '事件ID',
  `evt_cls` varchar(3) DEFAULT NULL COMMENT '事件分类',
  `alias_id` decimal(20,0) DEFAULT NULL COMMENT '别名ID',
  `effect_tm` timestamp NULL DEFAULT NULL COMMENT '生效时间',
  `sys_refer_id` decimal(20,0) DEFAULT NULL COMMENT '系统关联ID',
  `evt_loc` varchar(64) DEFAULT NULL COMMENT '事件位置（源发系统）',
  `noticed_tm` timestamp NULL DEFAULT NULL COMMENT '察觉时间',
  `evt_typ` decimal(20,0) DEFAULT NULL COMMENT '事件类型',
  `achv_prom_targ_id` decimal(20,0) DEFAULT NULL COMMENT '达成促销目标ID',
  `adjust_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '调整事件信息ID',
  `air_award_tkt_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '航空奖励机票事件信息ID',
  `air_award_cncl_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '航空奖励取消事件信息ID',
  `air_award_ugrd_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '航空奖励升舱事件信息ID',
  `award_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '奖励事件信息ID',
  `buy_mile_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '购买里程事件信息ID',
  `cncl_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '取消事件信息ID',
  `compen_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '补偿事件信息ID',
  `elite_lvl_bnft_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '精英级别收益事件信息ID',
  `gift_mile_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '奖励里程事件信息ID',
  `mamtax_evt_id` decimal(20,0) DEFAULT NULL COMMENT 'MAMTAX事件ID',
  `manual_pay_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '手工支付事件信息ID',
  `mbr_id` decimal(20,0) DEFAULT NULL COMMENT '会员ID',
  `prom_elite_lvl_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '促销精英级别事件信息ID',
  `prom_gift_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '促销奖励事件信息ID',
  `refer_evt_id` decimal(20,0) DEFAULT NULL COMMENT '关联事件ID',
  `sec_bnft_mbr_id` decimal(20,0) DEFAULT NULL COMMENT '第二受益会员ID',
  `sub_prog_bnft_evt_info_id` decimal(20,0) DEFAULT NULL COMMENT '子计划受益事件信息ID',
  `transfer_src_evt_id` decimal(20,0) DEFAULT NULL COMMENT '转换来源事件ID',
  `award_cd` varchar(20) DEFAULT NULL COMMENT '奖励代码',
  `chnl` varchar(10) DEFAULT NULL COMMENT '渠道',
  `external_cd` varchar(60) DEFAULT NULL COMMENT '外部代码（内部订单号）',
  `pnr` varchar(6) DEFAULT NULL COMMENT 'PNR',
  `ins_tm` timestamp NULL DEFAULT NULL COMMENT '插入时间',
  `ins_id` varchar(32) DEFAULT NULL COMMENT '插入人ID',
  `evt_sub_typ` varchar(20) DEFAULT NULL COMMENT '事件子类型',
  `operator` varchar(32) DEFAULT NULL COMMENT '操作人',
  `proc_dep` varchar(200) DEFAULT NULL COMMENT '处理单位',
  `expiry_time` timestamp NULL DEFAULT NULL,
  `redemption_policy` varchar(30) DEFAULT NULL,
  `legacy` decimal(5,2) DEFAULT NULL,
  `pay_id` decimal(22,2) DEFAULT NULL,
  `notes` varchar(260) DEFAULT NULL,
  `param1` varchar(100) DEFAULT NULL,
  `param2` varchar(100) DEFAULT NULL,
  `param3` varchar(100) DEFAULT NULL,
  `param4` varchar(100) DEFAULT NULL,
  `param5` varchar(100) DEFAULT NULL,
  `param6` varchar(100) DEFAULT NULL,
  `trsf_evt_info_id` decimal(22,2) DEFAULT NULL,
  `program_curr_beneent_info_id` decimal(22,2) DEFAULT NULL,
  `ins_dt` date DEFAULT NULL COMMENT '插入日期',
  `tx_date` date DEFAULT NULL COMMENT 'TX_DATE',
  `src_system` char(3) DEFAULT NULL COMMENT 'SRC_SYSTEM',
  `etl_time` timestamp NULL DEFAULT NULL COMMENT 'ETL_TIME'
) DEFAULT CHARSET = utf8mb4 ROW_FORMAT = DYNAMIC COMPRESSION = 'zstd_1.3.8' REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
 partition by range columns(Ins_Dt)
(partition M190001 values less than ('1900-02-01'),
partition M201201 values less than ('2012-02-01'),
partition M201202 values less than ('2012-03-01'),
partition M201203 values less than ('2012-04-01'),
partition M201204 values less than ('2012-05-01'),
partition M201205 values less than ('2012-06-01'),
partition M201206 values less than ('2012-07-01'),
partition M201207 values less than ('2012-08-01'),
partition M201208 values less than ('2012-09-01'),
partition M201209 values less than ('2012-10-01'),
partition M201210 values less than ('2012-11-01'),
partition M201211 values less than ('2012-12-01'),
partition M201212 values less than ('2013-01-01'),
partition M201301 values less than ('2013-02-01'),
partition M201302 values less than ('2013-03-01'),
partition M201303 values less than ('2013-04-01'),
partition M201304 values less than ('2013-05-01'),
partition M201305 values less than ('2013-06-01'),
partition M201306 values less than ('2013-07-01'),
partition M201307 values less than ('2013-08-01'),
partition M201308 values less than ('2013-09-01'),
partition M201309 values less than ('2013-10-01'),
partition M201310 values less than ('2013-11-01'),
partition M201311 values less than ('2013-12-01'),
partition M201312 values less than ('2014-01-01'),
partition M201401 values less than ('2014-02-01'),
partition M201402 values less than ('2014-03-01'),
partition M201403 values less than ('2014-04-01'),
partition M201404 values less than ('2014-05-01'),
partition M201405 values less than ('2014-06-01'),
partition M201406 values less than ('2014-07-01'),
partition M201407 values less than ('2014-08-01'),
partition M201408 values less than ('2014-09-01'),
partition M201409 values less than ('2014-10-01'),
partition M201410 values less than ('2014-11-01'),
partition M201411 values less than ('2014-12-01'),
partition M201412 values less than ('2015-01-01'),
partition M201501 values less than ('2015-02-01'),
partition M201502 values less than ('2015-03-01'),
partition M201503 values less than ('2015-04-01'),
partition M201504 values less than ('2015-05-01'),
partition M201505 values less than ('2015-06-01'),
partition M201506 values less than ('2015-07-01'),
partition M201507 values less than ('2015-08-01'),
partition M201508 values less than ('2015-09-01'),
partition M201509 values less than ('2015-10-01'),
partition M201510 values less than ('2015-11-01'),
partition M201511 values less than ('2015-12-01'),
partition M201512 values less than ('2016-01-01'),
partition M201601 values less than ('2016-02-01'),
partition M201602 values less than ('2016-03-01'),
partition M201603 values less than ('2016-04-01'),
partition M201604 values less than ('2016-05-01'),
partition M201605 values less than ('2016-06-01'),
partition M201606 values less than ('2016-07-01'),
partition M201607 values less than ('2016-08-01'),
partition M201608 values less than ('2016-09-01'),
partition M201609 values less than ('2016-10-01'),
partition M201610 values less than ('2016-11-01'),
partition M201611 values less than ('2016-12-01'),
partition M201612 values less than ('2017-01-01'),
partition M201701 values less than ('2017-02-01'),
partition M201702 values less than ('2017-03-01'),
partition M201703 values less than ('2017-04-01'),
partition M201704 values less than ('2017-05-01'),
partition M201705 values less than ('2017-06-01'),
partition M201706 values less than ('2017-07-01'),
partition M201707 values less than ('2017-08-01'),
partition M201708 values less than ('2017-09-01'),
partition M201709 values less than ('2017-10-01'),
partition M201710 values less than ('2017-11-01'),
partition M201711 values less than ('2017-12-01'),
partition M201712 values less than ('2018-01-01'),
partition M201801 values less than ('2018-02-01'),
partition M201802 values less than ('2018-03-01'),
partition M201803 values less than ('2018-04-01'),
partition M201804 values less than ('2018-05-01'),
partition M201805 values less than ('2018-06-01'),
partition M201806 values less than ('2018-07-01'),
partition M201807 values less than ('2018-08-01'),
partition M201808 values less than ('2018-09-01'),
partition M201809 values less than ('2018-10-01'),
partition M201810 values less than ('2018-11-01'),
partition M201811 values less than ('2018-12-01'),
partition M201812 values less than ('2019-01-01'),
partition M201901 values less than ('2019-02-01'),
partition M201902 values less than ('2019-03-01'),
partition M201903 values less than ('2019-04-01'),
partition M201904 values less than ('2019-05-01'),
partition M201905 values less than ('2019-06-01'),
partition M201906 values less than ('2019-07-01'),
partition M201907 values less than ('2019-08-01'),
partition M201908 values less than ('2019-09-01'),
partition M201909 values less than ('2019-10-01'),
partition M201910 values less than ('2019-11-01'),
partition M201911 values less than ('2019-12-01'),
partition M201912 values less than ('2020-01-01'),
partition M202001 values less than ('2020-02-01'),
partition M202002 values less than ('2020-03-01'),
partition M202003 values less than ('2020-04-01'),
partition M202004 values less than ('2020-05-01'),
partition M202005 values less than ('2020-06-01'),
partition M202006 values less than ('2020-07-01'),
partition M202007 values less than ('2020-08-01'),
partition M202008 values less than ('2020-09-01'),
partition M202009 values less than ('2020-10-01'),
partition M202010 values less than ('2020-11-01'),
partition M202011 values less than ('2020-12-01'),
partition M202012 values less than ('2021-01-01'),
partition M202101 values less than ('2021-02-01'),
partition M202102 values less than ('2021-03-01'),
partition M202103 values less than ('2021-04-01'),
partition M202104 values less than ('2021-05-01'),
partition M202105 values less than ('2021-06-01'),
partition M202106 values less than ('2021-07-01'),
partition M202107 values less than ('2021-08-01'),
partition M202108 values less than ('2021-09-01'),
partition M202109 values less than ('2021-10-01'),
partition M202110 values less than ('2021-11-01'),
partition M202111 values less than ('2021-12-01'),
partition M202112 values less than ('2022-01-01'),
partition M202201 values less than ('2022-02-01'),
partition M202202 values less than ('2022-03-01'),
partition M202203 values less than ('2022-04-01'),
partition M202204 values less than ('2022-05-01'),
partition M202205 values less than ('2022-06-01'),
partition M202206 values less than ('2022-07-01'),
partition M202207 values less than ('2022-08-01'),
partition M202208 values less than ('2022-09-01'),
partition M202209 values less than ('2022-10-01'),
partition M202210 values less than ('2022-11-01'),
partition M202211 values less than ('2022-12-01'),
partition M202212 values less than ('2023-01-01'),
partition M202301 values less than ('2023-02-01'),
partition M202302 values less than ('2023-03-01'),
partition M202303 values less than ('2023-04-01'),
partition M202304 values less than ('2023-05-01'),
partition M202305 values less than ('2023-06-01'),
partition M202306 values less than ('2023-07-01'),
partition M202307 values less than ('2023-08-01'),
partition M202308 values less than ('2023-09-01'),
partition M202309 values less than ('2023-10-01'),
partition M202310 values less than ('2023-11-01'),
partition M202311 values less than ('2023-12-01'),
partition M202312 values less than ('2024-01-01'),
partition M202401 values less than ('2024-02-01'),
partition M202402 values less than ('2024-03-01'),
partition M202403 values less than ('2024-04-01'),
partition M202404 values less than ('2024-05-01'),
partition M202405 values less than ('2024-06-01'),
partition M202406 values less than ('2024-07-01'),
partition M202407 values less than ('2024-08-01'),
partition M202408 values less than ('2024-09-01'),
partition M202409 values less than ('2024-10-01'),
partition M202410 values less than ('2024-11-01'),
partition M202411 values less than ('2024-12-01'),
partition M202412 values less than ('2025-01-01'),
partition M202501 values less than ('2025-02-01'),
partition M202502 values less than ('2025-03-01'),
partition M202503 values less than ('2025-04-01'),
partition M202504 values less than ('2025-05-01'),
partition M202505 values less than ('2025-06-01'),
partition M202506 values less than ('2025-07-01'),
partition M202507 values less than ('2025-08-01'),
partition M202508 values less than ('2025-09-01'),
partition M202509 values less than ('2025-10-01'),
partition M202510 values less than ('2025-11-01'),
partition M202511 values less than ('2025-12-01'),
partition M202512 values less than ('2026-01-01'),
partition M202601 values less than ('2026-02-01'),
partition M202602 values less than ('2026-03-01'),
partition M202603 values less than ('2026-04-01'),
partition M202604 values less than ('2026-05-01'),
partition M202605 values less than ('2026-06-01'),
partition M202606 values less than ('2026-07-01'),
partition M202607 values less than ('2026-08-01'),
partition M202608 values less than ('2026-09-01'),
partition M202609 values less than ('2026-10-01'),
partition M202610 values less than ('2026-11-01'),
partition M202611 values less than ('2026-12-01'),
partition M202612 values less than ('2027-01-01'),
partition M202701 values less than ('2027-02-01'),
partition M202702 values less than ('2027-03-01'),
partition M202703 values less than ('2027-04-01'),
partition M202704 values less than ('2027-05-01'),
partition M202705 values less than ('2027-06-01'),
partition M202706 values less than ('2027-07-01'),
partition M202707 values less than ('2027-08-01'),
partition M202708 values less than ('2027-09-01'),
partition M202709 values less than ('2027-10-01'),
partition M202710 values less than ('2027-11-01'),
partition M202711 values less than ('2027-12-01'),
partition M202712 values less than ('2028-01-01'),
partition M202801 values less than ('2028-02-01'),
partition M202802 values less than ('2028-03-01'),
partition M202803 values less than ('2028-04-01'),
partition M202804 values less than ('2028-05-01'),
partition M202805 values less than ('2028-06-01'),
partition M202806 values less than ('2028-07-01'),
partition M202807 values less than ('2028-08-01'),
partition M202808 values less than ('2028-09-01'),
partition M202809 values less than ('2028-10-01'),
partition M202810 values less than ('2028-11-01'),
partition M202811 values less than ('2028-12-01'),
partition M202812 values less than ('2029-01-01'),
partition M202901 values less than ('2029-02-01'),
partition M202902 values less than ('2029-03-01'),
partition M202903 values less than ('2029-04-01'),
partition M202904 values less than ('2029-05-01'),
partition M202905 values less than ('2029-06-01'),
partition M202906 values less than ('2029-07-01'),
partition M202907 values less than ('2029-08-01'),
partition M202908 values less than ('2029-09-01'),
partition M202909 values less than ('2029-10-01'),
partition M202910 values less than ('2029-11-01'),
partition M202911 values less than ('2029-12-01'),
partition M202912 values less than ('2030-01-01'),
partition M203001 values less than ('2030-02-01'),
partition M203002 values less than ('2030-03-01'),
partition M203003 values less than ('2030-04-01'),
partition M203004 values less than ('2030-05-01'),
partition M203005 values less than ('2030-06-01'),
partition M203006 values less than ('2030-07-01'),
partition M203007 values less than ('2030-08-01'),
partition M203008 values less than ('2030-09-01'),
partition M203009 values less than ('2030-10-01'),
partition M203010 values less than ('2030-11-01'),
partition M203011 values less than ('2030-12-01'),
partition M203012 values less than ('2031-01-01'),
partition M203101 values less than ('2031-02-01'),
partition M203102 values less than ('2031-03-01'),
partition M203103 values less than ('2031-04-01'),
partition M203104 values less than ('2031-05-01'),
partition M203105 values less than ('2031-06-01'),
partition M203106 values less than ('2031-07-01'),
partition M203107 values less than ('2031-08-01'),
partition M203108 values less than ('2031-09-01'),
partition M203109 values less than ('2031-10-01'),
partition M203110 values less than ('2031-11-01'),
partition M203111 values less than ('2031-12-01'),
partition M203112 values less than ('2032-01-01'),
partition M203201 values less than ('2032-02-01'),
partition M203202 values less than ('2032-03-01'),
partition M203203 values less than ('2032-04-01'),
partition M203204 values less than ('2032-05-01'),
partition M203205 values less than ('2032-06-01'),
partition M203206 values less than ('2032-07-01'),
partition M203207 values less than ('2032-08-01'),
partition M203208 values less than ('2032-09-01'),
partition M203209 values less than ('2032-10-01'),
partition M203210 values less than ('2032-11-01'),
partition M203211 values less than ('2032-12-01'),
partition M203212 values less than ('2033-01-01'),
partition M203301 values less than ('2033-02-01'),
partition M203302 values less than ('2033-03-01'),
partition M203303 values less than ('2033-04-01'),
partition M203304 values less than ('2033-05-01'),
partition M203305 values less than ('2033-06-01'),
partition M203306 values less than ('2033-07-01'),
partition M203307 values less than ('2033-08-01'),
partition M203308 values less than ('2033-09-01'),
partition M203309 values less than ('2033-10-01'),
partition M203310 values less than ('2033-11-01'),
partition M203311 values less than ('2033-12-01'),
partition M203312 values less than ('2034-01-01'),
partition M203401 values less than ('2034-02-01'),
partition M203402 values less than ('2034-03-01'),
partition M203403 values less than ('2034-04-01'),
partition M203404 values less than ('2034-05-01'),
partition M203405 values less than ('2034-06-01'),
partition M203406 values less than ('2034-07-01'),
partition M203407 values less than ('2034-08-01'),
partition M203408 values less than ('2034-09-01'),
partition M203409 values less than ('2034-10-01'),
partition M203410 values less than ('2034-11-01'),
partition M203411 values less than ('2034-12-01'),
partition M203412 values less than ('2035-01-01'),
partition M203501 values less than ('2035-02-01'),
partition M203502 values less than ('2035-03-01'),
partition M203503 values less than ('2035-04-01'),
partition M203504 values less than ('2035-05-01'),
partition M203505 values less than ('2035-06-01'),
partition M203506 values less than ('2035-07-01'),
partition M203507 values less than ('2035-08-01'),
partition M203508 values less than ('2035-09-01'),
partition M203509 values less than ('2035-10-01'),
partition M203510 values less than ('2035-11-01'),
partition M203511 values less than ('2035-12-01'),
partition M203512 values less than ('2036-01-01'),
partition M203601 values less than ('2036-02-01'),
partition M203602 values less than ('2036-03-01'),
partition M203603 values less than ('2036-04-01'),
partition M203604 values less than ('2036-05-01'),
partition M203605 values less than ('2036-06-01'),
partition M203606 values less than ('2036-07-01'),
partition M203607 values less than ('2036-08-01'),
partition M203608 values less than ('2036-09-01'),
partition M203609 values less than ('2036-10-01'),
partition M203610 values less than ('2036-11-01'),
partition M203611 values less than ('2036-12-01'),
partition M203612 values less than ('2037-01-01'),
partition M203701 values less than ('2037-02-01'),
partition M203702 values less than ('2037-03-01'),
partition M203703 values less than ('2037-04-01'),
partition M203704 values less than ('2037-05-01'),
partition M203705 values less than ('2037-06-01'),
partition M203706 values less than ('2037-07-01'),
partition M203707 values less than ('2037-08-01'),
partition M203708 values less than ('2037-09-01'),
partition M203709 values less than ('2037-10-01'),
partition M203710 values less than ('2037-11-01'),
partition M203711 values less than ('2037-12-01'),
partition M203712 values less than ('2038-01-01'),
partition M203801 values less than ('2038-02-01'),
partition M203802 values less than ('2038-03-01'),
partition M203803 values less than ('2038-04-01'),
partition M203804 values less than ('2038-05-01'),
partition M203805 values less than ('2038-06-01'),
partition M203806 values less than ('2038-07-01'),
partition M203807 values less than ('2038-08-01'),
partition M203808 values less than ('2038-09-01'),
partition M203809 values less than ('2038-10-01'),
partition M203810 values less than ('2038-11-01'),
partition M203811 values less than ('2038-12-01'),
partition M203812 values less than ('2039-01-01'),
partition M203901 values less than ('2039-02-01'),
partition M203902 values less than ('2039-03-01'),
partition M203903 values less than ('2039-04-01'),
partition M203904 values less than ('2039-05-01'),
partition M203905 values less than ('2039-06-01'),
partition M203906 values less than ('2039-07-01'),
partition M203907 values less than ('2039-08-01'),
partition M203908 values less than ('2039-09-01'),
partition M203909 values less than ('2039-10-01'),
partition M203910 values less than ('2039-11-01'),
partition M203911 values less than ('2039-12-01'),
partition M203912 values less than ('2040-01-01'),
partition M204001 values less than ('2040-02-01'),
partition M204002 values less than ('2040-03-01'),
partition M204003 values less than ('2040-04-01'),
partition M204004 values less than ('2040-05-01'),
partition M204005 values less than ('2040-06-01'),
partition M204006 values less than ('2040-07-01'),
partition M204007 values less than ('2040-08-01'),
partition M204008 values less than ('2040-09-01'),
partition M204009 values less than ('2040-10-01'),
partition M204010 values less than ('2040-11-01'),
partition M204011 values less than ('2040-12-01'),
partition MMAX values less than (MAXVALUE)) |

| act_membership | CREATE TABLE `act_membership` (
  `mbr_id` decimal(20,0) DEFAULT NULL COMMENT '会员id',
  `startdt` date DEFAULT NULL COMMENT 'start dt',
  `enddt` date DEFAULT NULL COMMENT 'end dt',
  `acq_partner` varchar(10) DEFAULT NULL COMMENT '获得合作伙伴',
  `vip_lvl_rsn` varchar(20) DEFAULT NULL COMMENT 'vip级别原因',
  `block_sts` varchar(20) DEFAULT NULL COMMENT '锁定状态',
  `bonus_acct_id` decimal(20,0) DEFAULT NULL COMMENT '奖励账号id',
  `cust_id` decimal(20,0) DEFAULT NULL COMMENT '客户id',
  `cust_val_score` decimal(20,0) DEFAULT NULL COMMENT '客户价值得分',
  `elite_acct_id` decimal(20,0) DEFAULT NULL COMMENT '精英账户id',
  `enroll_dt` date DEFAULT NULL COMMENT '注册日期',
  `enroll_chnl_cd` varchar(7) DEFAULT NULL COMMENT '注册渠道代码',
  `fraud_score` decimal(20,0) DEFAULT NULL COMMENT '欺诈评分',
  `fraud_upd_dt` timestamp NULL DEFAULT NULL COMMENT '欺诈评分修改日期',
  `fraud_upd_id` varchar(30) DEFAULT NULL COMMENT '欺诈评分修改人id',
  `inact_dt` date DEFAULT NULL COMMENT '失效日期',
  `test_ind` decimal(20,0) DEFAULT NULL COMMENT '测试标识',
  `last_fraud_suspect_dt` date DEFAULT NULL COMMENT '上次欺诈嫌疑日期',
  `max_elite_lvl_beg_dt` date DEFAULT NULL COMMENT '最高精英级别开始日期',
  `max_elite_lvl_end_dt` date DEFAULT NULL COMMENT '最高精英级别结束日期',
  `max_lvl` varchar(20) DEFAULT NULL COMMENT '最高级别',
  `miss_match_cnt` int(11) DEFAULT NULL COMMENT '不匹配次数',
  `nm_on_card` varchar(27) DEFAULT NULL COMMENT '卡面姓名',
  `nm_on_card_manual_upd_ind` int(11) DEFAULT NULL COMMENT '卡面姓名手工修改标识',
  `pin` varchar(256) DEFAULT NULL COMMENT 'pin',
  `mbr_role` int(11) DEFAULT NULL COMMENT '角色（1=owner 2=collector 3=spender）',
  `vip_sponsor` varchar(20) DEFAULT NULL COMMENT 'vip发起人',
  `ins_id` varchar(30) DEFAULT NULL COMMENT '插入人id',
  `upd_id` varchar(30) DEFAULT NULL COMMENT '修改人id',
  `promoter_id` decimal(20,0) DEFAULT NULL COMMENT '介绍人id',
  `ins_tm` timestamp NULL DEFAULT NULL COMMENT '插入时间',
  `ins_dt` date DEFAULT NULL COMMENT '插入日期',
  `tx_date` date DEFAULT NULL COMMENT 'tx_date',
  `src_system` char(3) DEFAULT NULL COMMENT 'src_system',
  `etl_time` timestamp NULL DEFAULT NULL COMMENT 'etl_time'
) DEFAULT CHARSET = utf8mb4 ROW_FORMAT = DYNAMIC COMPRESSION = 'zstd_1.3.8' REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0
 partition by range columns(Ins_Dt)
(partition M190001 values less than ('1900-02-01'),
partition M201201 values less than ('2012-02-01'),
partition M201202 values less than ('2012-03-01'),
partition M201203 values less than ('2012-04-01'),
partition M201204 values less than ('2012-05-01'),
partition M201205 values less than ('2012-06-01'),
partition M201206 values less than ('2012-07-01'),
partition M201207 values less than ('2012-08-01'),
partition M201208 values less than ('2012-09-01'),
partition M201209 values less than ('2012-10-01'),
partition M201210 values less than ('2012-11-01'),
partition M201211 values less than ('2012-12-01'),
partition M201212 values less than ('2013-01-01'),
partition M201301 values less than ('2013-02-01'),
partition M201302 values less than ('2013-03-01'),
partition M201303 values less than ('2013-04-01'),
partition M201304 values less than ('2013-05-01'),
partition M201305 values less than ('2013-06-01'),
partition M201306 values less than ('2013-07-01'),
partition M201307 values less than ('2013-08-01'),
partition M201308 values less than ('2013-09-01'),
partition M201309 values less than ('2013-10-01'),
partition M201310 values less than ('2013-11-01'),
partition M201311 values less than ('2013-12-01'),
partition M201312 values less than ('2014-01-01'),
partition M201401 values less than ('2014-02-01'),
partition M201402 values less than ('2014-03-01'),
partition M201403 values less than ('2014-04-01'),
partition M201404 values less than ('2014-05-01'),
partition M201405 values less than ('2014-06-01'),
partition M201406 values less than ('2014-07-01'),
partition M201407 values less than ('2014-08-01'),
partition M201408 values less than ('2014-09-01'),
partition M201409 values less than ('2014-10-01'),
partition M201410 values less than ('2014-11-01'),
partition M201411 values less than ('2014-12-01'),
partition M201412 values less than ('2015-01-01'),
partition M201501 values less than ('2015-02-01'),
partition M201502 values less than ('2015-03-01'),
partition M201503 values less than ('2015-04-01'),
partition M201504 values less than ('2015-05-01'),
partition M201505 values less than ('2015-06-01'),
partition M201506 values less than ('2015-07-01'),
partition M201507 values less than ('2015-08-01'),
partition M201508 values less than ('2015-09-01'),
partition M201509 values less than ('2015-10-01'),
partition M201510 values less than ('2015-11-01'),
partition M201511 values less than ('2015-12-01'),
partition M201512 values less than ('2016-01-01'),
partition M201601 values less than ('2016-02-01'),
partition M201602 values less than ('2016-03-01'),
partition M201603 values less than ('2016-04-01'),
partition M201604 values less than ('2016-05-01'),
partition M201605 values less than ('2016-06-01'),
partition M201606 values less than ('2016-07-01'),
partition M201607 values less than ('2016-08-01'),
partition M201608 values less than ('2016-09-01'),
partition M201609 values less than ('2016-10-01'),
partition M201610 values less than ('2016-11-01'),
partition M201611 values less than ('2016-12-01'),
partition M201612 values less than ('2017-01-01'),
partition M201701 values less than ('2017-02-01'),
partition M201702 values less than ('2017-03-01'),
partition M201703 values less than ('2017-04-01'),
partition M201704 values less than ('2017-05-01'),
partition M201705 values less than ('2017-06-01'),
partition M201706 values less than ('2017-07-01'),
partition M201707 values less than ('2017-08-01'),
partition M201708 values less than ('2017-09-01'),
partition M201709 values less than ('2017-10-01'),
partition M201710 values less than ('2017-11-01'),
partition M201711 values less than ('2017-12-01'),
partition M201712 values less than ('2018-01-01'),
partition M201801 values less than ('2018-02-01'),
partition M201802 values less than ('2018-03-01'),
partition M201803 values less than ('2018-04-01'),
partition M201804 values less than ('2018-05-01'),
partition M201805 values less than ('2018-06-01'),
partition M201806 values less than ('2018-07-01'),
partition M201807 values less than ('2018-08-01'),
partition M201808 values less than ('2018-09-01'),
partition M201809 values less than ('2018-10-01'),
partition M201810 values less than ('2018-11-01'),
partition M201811 values less than ('2018-12-01'),
partition M201812 values less than ('2019-01-01'),
partition M201901 values less than ('2019-02-01'),
partition M201902 values less than ('2019-03-01'),
partition M201903 values less than ('2019-04-01'),
partition M201904 values less than ('2019-05-01'),
partition M201905 values less than ('2019-06-01'),
partition M201906 values less than ('2019-07-01'),
partition M201907 values less than ('2019-08-01'),
partition M201908 values less than ('2019-09-01'),
partition M201909 values less than ('2019-10-01'),
partition M201910 values less than ('2019-11-01'),
partition M201911 values less than ('2019-12-01'),
partition M201912 values less than ('2020-01-01'),
partition M202001 values less than ('2020-02-01'),
partition M202002 values less than ('2020-03-01'),
partition M202003 values less than ('2020-04-01'),
partition M202004 values less than ('2020-05-01'),
partition M202005 values less than ('2020-06-01'),
partition M202006 values less than ('2020-07-01'),
partition M202007 values less than ('2020-08-01'),
partition M202008 values less than ('2020-09-01'),
partition M202009 values less than ('2020-10-01'),
partition M202010 values less than ('2020-11-01'),
partition M202011 values less than ('2020-12-01'),
partition M202012 values less than ('2021-01-01'),
partition M202101 values less than ('2021-02-01'),
partition M202102 values less than ('2021-03-01'),
partition M202103 values less than ('2021-04-01'),
partition M202104 values less than ('2021-05-01'),
partition M202105 values less than ('2021-06-01'),
partition M202106 values less than ('2021-07-01'),
partition M202107 values less than ('2021-08-01'),
partition M202108 values less than ('2021-09-01'),
partition M202109 values less than ('2021-10-01'),
partition M202110 values less than ('2021-11-01'),
partition M202111 values less than ('2021-12-01'),
partition M202112 values less than ('2022-01-01'),
partition M202201 values less than ('2022-02-01'),
partition M202202 values less than ('2022-03-01'),
partition M202203 values less than ('2022-04-01'),
partition M202204 values less than ('2022-05-01'),
partition M202205 values less than ('2022-06-01'),
partition M202206 values less than ('2022-07-01'),
partition M202207 values less than ('2022-08-01'),
partition M202208 values less than ('2022-09-01'),
partition M202209 values less than ('2022-10-01'),
partition M202210 values less than ('2022-11-01'),
partition M202211 values less than ('2022-12-01'),
partition M202212 values less than ('2023-01-01'),
partition M202301 values less than ('2023-02-01'),
partition M202302 values less than ('2023-03-01'),
partition M202303 values less than ('2023-04-01'),
partition M202304 values less than ('2023-05-01'),
partition M202305 values less than ('2023-06-01'),
partition M202306 values less than ('2023-07-01'),
partition M202307 values less than ('2023-08-01'),
partition M202308 values less than ('2023-09-01'),
partition M202309 values less than ('2023-10-01'),
partition M202310 values less than ('2023-11-01'),
partition M202311 values less than ('2023-12-01'),
partition M202312 values less than ('2024-01-01'),
partition M202401 values less than ('2024-02-01'),
partition M202402 values less than ('2024-03-01'),
partition M202403 values less than ('2024-04-01'),
partition M202404 values less than ('2024-05-01'),
partition M202405 values less than ('2024-06-01'),
partition M202406 values less than ('2024-07-01'),
partition M202407 values less than ('2024-08-01'),
partition M202408 values less than ('2024-09-01'),
partition M202409 values less than ('2024-10-01'),
partition M202410 values less than ('2024-11-01'),
partition M202411 values less than ('2024-12-01'),
partition M202412 values less than ('2025-01-01'),
partition M202501 values less than ('2025-02-01'),
partition M202502 values less than ('2025-03-01'),
partition M202503 values less than ('2025-04-01'),
partition M202504 values less than ('2025-05-01'),
partition M202505 values less than ('2025-06-01'),
partition M202506 values less than ('2025-07-01'),
partition M202507 values less than ('2025-08-01'),
partition M202508 values less than ('2025-09-01'),
partition M202509 values less than ('2025-10-01'),
partition M202510 values less than ('2025-11-01'),
partition M202511 values less than ('2025-12-01'),
partition M202512 values less than ('2026-01-01'),
partition M202601 values less than ('2026-02-01'),
partition M202602 values less than ('2026-03-01'),
partition M202603 values less than ('2026-04-01'),
partition M202604 values less than ('2026-05-01'),
partition M202605 values less than ('2026-06-01'),
partition M202606 values less than ('2026-07-01'),
partition M202607 values less than ('2026-08-01'),
partition M202608 values less than ('2026-09-01'),
partition M202609 values less than ('2026-10-01'),
partition M202610 values less than ('2026-11-01'),
partition M202611 values less than ('2026-12-01'),
partition M202612 values less than ('2027-01-01'),
partition M202701 values less than ('2027-02-01'),
partition M202702 values less than ('2027-03-01'),
partition M202703 values less than ('2027-04-01'),
partition M202704 values less than ('2027-05-01'),
partition M202705 values less than ('2027-06-01'),
partition M202706 values less than ('2027-07-01'),
partition M202707 values less than ('2027-08-01'),
partition M202708 values less than ('2027-09-01'),
partition M202709 values less than ('2027-10-01'),
partition M202710 values less than ('2027-11-01'),
partition M202711 values less than ('2027-12-01'),
partition M202712 values less than ('2028-01-01'),
partition M202801 values less than ('2028-02-01'),
partition M202802 values less than ('2028-03-01'),
partition M202803 values less than ('2028-04-01'),
partition M202804 values less than ('2028-05-01'),
partition M202805 values less than ('2028-06-01'),
partition M202806 values less than ('2028-07-01'),
partition M202807 values less than ('2028-08-01'),
partition M202808 values less than ('2028-09-01'),
partition M202809 values less than ('2028-10-01'),
partition M202810 values less than ('2028-11-01'),
partition M202811 values less than ('2028-12-01'),
partition M202812 values less than ('2029-01-01'),
partition M202901 values less than ('2029-02-01'),
partition M202902 values less than ('2029-03-01'),
partition M202903 values less than ('2029-04-01'),
partition M202904 values less than ('2029-05-01'),
partition M202905 values less than ('2029-06-01'),
partition M202906 values less than ('2029-07-01'),
partition M202907 values less than ('2029-08-01'),
partition M202908 values less than ('2029-09-01'),
partition M202909 values less than ('2029-10-01'),
partition M202910 values less than ('2029-11-01'),
partition M202911 values less than ('2029-12-01'),
partition M202912 values less than ('2030-01-01'),
partition M203001 values less than ('2030-02-01'),
partition M203002 values less than ('2030-03-01'),
partition M203003 values less than ('2030-04-01'),
partition M203004 values less than ('2030-05-01'),
partition M203005 values less than ('2030-06-01'),
partition M203006 values less than ('2030-07-01'),
partition M203007 values less than ('2030-08-01'),
partition M203008 values less than ('2030-09-01'),
partition M203009 values less than ('2030-10-01'),
partition M203010 values less than ('2030-11-01'),
partition M203011 values less than ('2030-12-01'),
partition M203012 values less than ('2031-01-01'),
partition M203101 values less than ('2031-02-01'),
partition M203102 values less than ('2031-03-01'),
partition M203103 values less than ('2031-04-01'),
partition M203104 values less than ('2031-05-01'),
partition M203105 values less than ('2031-06-01'),
partition M203106 values less than ('2031-07-01'),
partition M203107 values less than ('2031-08-01'),
partition M203108 values less than ('2031-09-01'),
partition M203109 values less than ('2031-10-01'),
partition M203110 values less than ('2031-11-01'),
partition M203111 values less than ('2031-12-01'),
partition M203112 values less than ('2032-01-01'),
partition M203201 values less than ('2032-02-01'),
partition M203202 values less than ('2032-03-01'),
partition M203203 values less than ('2032-04-01'),
partition M203204 values less than ('2032-05-01'),
partition M203205 values less than ('2032-06-01'),
partition M203206 values less than ('2032-07-01'),
partition M203207 values less than ('2032-08-01'),
partition M203208 values less than ('2032-09-01'),
partition M203209 values less than ('2032-10-01'),
partition M203210 values less than ('2032-11-01'),
partition M203211 values less than ('2032-12-01'),
partition M203212 values less than ('2033-01-01'),
partition M203301 values less than ('2033-02-01'),
partition M203302 values less than ('2033-03-01'),
partition M203303 values less than ('2033-04-01'),
partition M203304 values less than ('2033-05-01'),
partition M203305 values less than ('2033-06-01'),
partition M203306 values less than ('2033-07-01'),
partition M203307 values less than ('2033-08-01'),
partition M203308 values less than ('2033-09-01'),
partition M203309 values less than ('2033-10-01'),
partition M203310 values less than ('2033-11-01'),
partition M203311 values less than ('2033-12-01'),
partition M203312 values less than ('2034-01-01'),
partition M203401 values less than ('2034-02-01'),
partition M203402 values less than ('2034-03-01'),
partition M203403 values less than ('2034-04-01'),
partition M203404 values less than ('2034-05-01'),
partition M203405 values less than ('2034-06-01'),
partition M203406 values less than ('2034-07-01'),
partition M203407 values less than ('2034-08-01'),
partition M203408 values less than ('2034-09-01'),
partition M203409 values less than ('2034-10-01'),
partition M203410 values less than ('2034-11-01'),
partition M203411 values less than ('2034-12-01'),
partition M203412 values less than ('2035-01-01'),
partition M203501 values less than ('2035-02-01'),
partition M203502 values less than ('2035-03-01'),
partition M203503 values less than ('2035-04-01'),
partition M203504 values less than ('2035-05-01'),
partition M203505 values less than ('2035-06-01'),
partition M203506 values less than ('2035-07-01'),
partition M203507 values less than ('2035-08-01'),
partition M203508 values less than ('2035-09-01'),
partition M203509 values less than ('2035-10-01'),
partition M203510 values less than ('2035-11-01'),
partition M203511 values less than ('2035-12-01'),
partition M203512 values less than ('2036-01-01'),
partition M203601 values less than ('2036-02-01'),
partition M203602 values less than ('2036-03-01'),
partition M203603 values less than ('2036-04-01'),
partition M203604 values less than ('2036-05-01'),
partition M203605 values less than ('2036-06-01'),
partition M203606 values less than ('2036-07-01'),
partition M203607 values less than ('2036-08-01'),
partition M203608 values less than ('2036-09-01'),
partition M203609 values less than ('2036-10-01'),
partition M203610 values less than ('2036-11-01'),
partition M203611 values less than ('2036-12-01'),
partition M203612 values less than ('2037-01-01'),
partition M203701 values less than ('2037-02-01'),
partition M203702 values less than ('2037-03-01'),
partition M203703 values less than ('2037-04-01'),
partition M203704 values less than ('2037-05-01'),
partition M203705 values less than ('2037-06-01'),
partition M203706 values less than ('2037-07-01'),
partition M203707 values less than ('2037-08-01'),
partition M203708 values less than ('2037-09-01'),
partition M203709 values less than ('2037-10-01'),
partition M203710 values less than ('2037-11-01'),
partition M203711 values less than ('2037-12-01'),
partition M203712 values less than ('2038-01-01'),
partition M203801 values less than ('2038-02-01'),
partition M203802 values less than ('2038-03-01'),
partition M203803 values less than ('2038-04-01'),
partition M203804 values less than ('2038-05-01'),
partition M203805 values less than ('2038-06-01'),
partition M203806 values less than ('2038-07-01'),
partition M203807 values less than ('2038-08-01'),
partition M203808 values less than ('2038-09-01'),
partition M203809 values less than ('2038-10-01'),
partition M203810 values less than ('2038-11-01'),
partition M203811 values less than ('2038-12-01'),
partition M203812 values less than ('2039-01-01'),
partition M203901 values less than ('2039-02-01'),
partition M203902 values less than ('2039-03-01'),
partition M203903 values less than ('2039-04-01'),
partition M203904 values less than ('2039-05-01'),
partition M203905 values less than ('2039-06-01'),
partition M203906 values less than ('2039-07-01'),
partition M203907 values less than ('2039-08-01'),
partition M203908 values less than ('2039-09-01'),
partition M203909 values less than ('2039-10-01'),
partition M203910 values less than ('2039-11-01'),
partition M203911 values less than ('2039-12-01'),
partition M203912 values less than ('2040-01-01'),
partition M204001 values less than ('2040-02-01'),
partition M204002 values less than ('2040-03-01'),
partition M204003 values less than ('2040-04-01'),
partition M204004 values less than ('2040-05-01'),
partition M204005 values less than ('2040-06-01'),
partition M204006 values less than ('2040-07-01'),
partition M204007 values less than ('2040-08-01'),
partition M204008 values less than ('2040-09-01'),
partition M204009 values less than ('2040-10-01'),
partition M204010 values less than ('2040-11-01'),
partition M204011 values less than ('2040-12-01'),
partition MMAX values less than (MAXVALUE)) |
+----------------+----------------------------------

| act_mileage_balance_txn_id | CREATE TABLE `act_mileage_balance_txn_id` (
  `TXN_ID` decimal(20,0) DEFAULT NULL
) DEFAULT CHARSET = utf8mb4 ROW_FORMAT = DYNAMIC COMPRESSION = 'zstd_1.3.8' REPLICA_NUM = 3 BLOCK_SIZE = 16384 USE_BLOOM_FILTER = FALSE TABLET_SIZE = 134217728 PCTFREE = 0 |
``` 

# SQL

```
SELECT /*+parallel（32） */    
	DISTINCT
	D.MBR_ID
FROM dev_edw.ACT_TRANSACTION A
    INNER JOIN dev_edw.ACT_EVENT C
        ON A.EVT_ID = C.EVT_ID
    INNER JOIN dev_edw.ACT_MEMBERSHIP D   --  /*保证是仍然存在的MBR_ID,所以仍然保留这一关联*/
        ON D.ENDDT = CAST('29991231' AS DATE)
            AND C.MBR_ID = D.MBR_ID
WHERE A.TXN_TYP = 1   --  /*以入账记录为基础*/
AND EXISTS (SELECT 1 FROM dev_tmp.ACT_MILEAGE_BALANCE_TXN_ID T WHERE T.TXN_ID=A.TXN_ID)
;
``` 

# 执行计划

### OB 3.2

```
-------OB 3.2 版本
| ===============================================================================
| ID | OPERATOR | NAME | EST. ROWS | COST |
| --- | --- | --- | --- | --- |
| 0 | PX COORDINATOR |  | 88077 | 1103067494 |
| 1 | EXCHANGE OUT DISTR | :EX10005 | 88077 | 1103006027 |
| 2 | HASH DISTINCT |  | 88077 | 1103006027 |
| 3 | EXCHANGE IN DISTR |  | 88077 | 1102923188 |
| 4 | EXCHANGE OUT DISTR (HASH) | :EX10004 | 88077 | 1102861722 |
| 5 | HASH DISTINCT |  | 88077 | 1102861722 |
| 6 | HASH RIGHT SEMI JOIN |  | 88090 | 1102778883 |
| 7 | EXCHANGE IN DISTR |  | 1010948 | 485109 |
| 8 | EXCHANGE OUT DISTR (BROADCAST) | :EX10000 | 1010948 | 391040 |
| 9 | PX BLOCK ITERATOR |  | 1010948 | 391040 |
| 10 | TABLE SCAN | T | 1010948 | 391040 |
| 11 | HASH JOIN |  | 16130215 | 1095023745 |
| 12 | JOIN FILTER CREATE |  | 3524299 | 360614547 |
| 13 | EXCHANGE IN DISTR |  | 3524299 | 360614547 |
| 14 | EXCHANGE OUT DISTR (HASH) | :EX10002 | 3524299 | 359138836 |
| 15 | HASH JOIN |  | 3524299 | 359138836 |
| 16 | JOIN FILTER CREATE |  | 135082 | 31435298 |
| 17 | EXCHANGE IN DISTR |  | 135082 | 31435298 |
| 18 | EXCHANGE OUT DISTR (BROADCAST) | :EX10001 | 135082 | 31410160 |
| 19 | PX BLOCK ITERATOR |  | 135082 | 31410160 |
| 20 | TABLE SCAN | D | 135082 | 31410160 |
| 21 | JOIN FILTER USE |  | 796525373 | 308099758 |
| 22 | PX BLOCK ITERATOR |  | 796525373 | 308099758 |
| 23 | TABLE SCAN | C | 796525373 | 308099758 |
| 24 | EXCHANGE IN DISTR |  | 646601861 | 707992692 |
| 25 | EXCHANGE OUT DISTR (HASH) | :EX10003 | 646601861 | 527493854 |
| 26 | JOIN FILTER USE |  | 646601861 | 527493854 |
| 27 | PX BLOCK ITERATOR |  | 646601861 | 527493854 |
| 28 | TABLE SCAN | A | 646601861 | 527493854 | ===============================================================================

Outputs & filters: 
-------------------------------------
  0 - output([INTERNAL_FUNCTION(D.mbr_id(0xffc93349d000))(0xffa7246897d0)]), filter(nil), rowset=256
  1 - output([INTERNAL_FUNCTION(D.mbr_id(0xffc93349d000))(0xffa7246897d0)]), filter(nil), rowset=256, dop=32
  2 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256, 
      distinct([D.mbr_id(0xffc93349d000)])
  3 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256
  4 - (#keys=1, [D.mbr_id(0xffc93349d000)]), output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256, dop=32
  5 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256, 
      distinct([D.mbr_id(0xffc93349d000)]), block
  6 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256, 
      equal_conds([T.TXN_ID(0xffc9334e6390) = A.txn_id(0xffc9334e6690)(0xffc9334f1b70)]), other_conds(nil)
  7 - output([T.TXN_ID(0xffc9334e6390)]), filter(nil), rowset=256
  8 - output([T.TXN_ID(0xffc9334e6390)]), filter(nil), rowset=256, dop=32
  9 - output([T.TXN_ID(0xffc9334e6390)]), filter(nil), rowset=256
  10 - output([T.TXN_ID(0xffc9334e6390)]), filter(nil), rowset=256, 
      access([T.TXN_ID(0xffc9334e6390)]), partitions(p0), 
      is_index_back=false, 
      range_key([T.__pk_increment(0xffc9335b51a0)]), range(MIN ; MAX)always true
  11 - output([D.mbr_id(0xffc93349d000)], [A.txn_id(0xffc9334e6690)]), filter(nil), rowset=256, 
      equal_conds([A.evt_id(0xffc933498e20) = C.evt_id(0xffc933499120)(0xffc933498700)]), other_conds(nil)
  12 - output([D.mbr_id(0xffc93349d000)], [C.evt_id(0xffc933499120)]), filter(nil), rowset=256
  13 - output([D.mbr_id(0xffc93349d000)], [C.evt_id(0xffc933499120)]), filter(nil), rowset=256
  14 - (#keys=1, [C.evt_id(0xffc933499120)]), output([D.mbr_id(0xffc93349d000)], [C.evt_id(0xffc933499120)]), filter(nil), rowset=256, dop=32
  15 - output([D.mbr_id(0xffc93349d000)], [C.evt_id(0xffc933499120)]), filter(nil), rowset=256, 
      equal_conds([C.mbr_id(0xffc93349cd00) = D.mbr_id(0xffc93349d000)(0xffc93349c5e0)]), other_conds(nil)
  16 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256
  17 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256
  18 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256, dop=32
  19 - output([D.mbr_id(0xffc93349d000)]), filter(nil), rowset=256, 
      asc.
  20 - output([D.mbr_id(0xffc93349d000)]), filter([D.enddt(0xffc93349bb00) = ?(0xffc93349b3a0)]), rowset=256, 
      access([D.enddt(0xffc93349bb00)], [D.mbr_id(0xffc93349d000)]), partitions(p[0-60]), 
      is_index_back=false, filter_before_indexback[false], 
      range_key([D.ins_dt(0xffc93349a000)], [D.__pk_increment(0xffc9335ab840)]), range(MIN,MIN ; MAX,MAX)always true
  21 - output([C.evt_id(0xffc933499120)], [C.mbr_id(0xffc93349cd00)]), filter(nil), rowset=256
  22 - output([C.evt_id(0xffc933499120)], [C.mbr_id(0xffc93349cd00)]), filter(nil), rowset=256, 
      asc.
  23 - output([C.evt_id(0xffc933499120)], [C.mbr_id(0xffc93349cd00)]), filter([SYS_OP_BLOOM_FILTER(C.mbr_id(0xffc93349cd00))(0xffa724680cf0)]), rowset=256, 
      access([C.evt_id(0xffc933499120)], [C.mbr_id(0xffc93349cd00)]), partitions(p[0-60]), 
      is_index_back=false, filter_before_indexback[false], 
      range_key([C.ins_dt(0xffc933497da0)], [C.__pk_increment(0xffc933588e30)]), range(MIN,MIN ; MAX,MAX)always true
  24 - output([A.evt_id(0xffc933498e20)], [A.txn_id(0xffc9334e6690)]), filter(nil), rowset=256
  25 - (#keys=1, [A.evt_id(0xffc933498e20)]), output([A.evt_id(0xffc933498e20)], [A.txn_id(0xffc9334e6690)]), filter(nil), rowset=256, dop=32
  26 - output([A.evt_id(0xffc933498e20)], [A.txn_id(0xffc9334e6690)]), filter(nil), rowset=256
  27 - output([A.evt_id(0xffc933498e20)], [A.txn_id(0xffc9334e6690)]), filter(nil), rowset=256, 
      asc.
  28 - output([A.evt_id(0xffc933498e20)], [A.txn_id(0xffc9334e6690)]), filter([SYS_OP_BLOOM_FILTER(A.evt_id(0xffc933498e20))(0xffa724683ee0)], [A.txn_typ(0xffc93349e0b0) = ?(0xffc93349d990)]), rowset=256, 
      access([A.evt_id(0xffc933498e20)], [A.txn_typ(0xffc93349e0b0)], [A.txn_id(0xffc9334e6690)]), partitions(p[0-60]), 
      is_index_back=false, filter_before_indexback[false,false], 
      range_key([A.ins_dt(0xffc933496f80)], [A.__pk_increment(0xffc93355f760)]), range(MIN,MIN ; MAX,MAX)always true

Used Hint:
-------------------------------------
  /*+
      PARALLEL(32)
  */

Outline Data:
-------------------------------------
  /*+
      BEGIN_OUTLINE_DATA
      USE_HASH_AGGREGATION(@"SEL$1")
      LEADING(@"SEL$1" ("dev_tmp.T"@"SEL$1" (("dev_edw.D"@"SEL$1" "dev_edw.C"@"SEL$1" )"dev_edw.A"@"SEL$1" )))
      USE_HASH(@"SEL$1" ("dev_edw.D"@"SEL$1" "dev_edw.C"@"SEL$1" "dev_edw.A"@"SEL$1" ))
      PQ_DISTRIBUTE(@"SEL$1" ("dev_edw.D"@"SEL$1" "dev_edw.C"@"SEL$1" "dev_edw.A"@"SEL$1" ) BROADCAST NONE)
      FULL(@"SEL$1" "dev_tmp.T"@"SEL$1")
      USE_HASH(@"SEL$1" ("dev_edw.A"@"SEL$1" ))
      PQ_DISTRIBUTE(@"SEL$1" ("dev_edw.A"@"SEL$1" ) HASH HASH)
      USE_HASH(@"SEL$1" ("dev_edw.C"@"SEL$1" ))
      PQ_DISTRIBUTE(@"SEL$1" ("dev_edw.C"@"SEL$1" ) BROADCAST NONE)
      FULL(@"SEL$1" "dev_edw.D"@"SEL$1")
      PX_JOIN_FILTER(@"SEL$1" ("dev_edw.C"@"SEL$1" ))
      FULL(@"SEL$1" "dev_edw.C"@"SEL$1")
      PX_JOIN_FILTER(@"SEL$1" ("dev_edw.A"@"SEL$1" ))
      FULL(@"SEL$1" "dev_edw.A"@"SEL$1")
      PARALLEL(32)
      END_OUTLINE_DATA
  */

Plan Type:
-------------------------------------
DISTRIBUTED

Optimization Info:
-------------------------------------
T:table_rows:1010948, physical_range_rows:1010948, logical_range_rows:1010948, index_back_rows:0, output_rows:1010948, est_method:local_storage, optimization_method=cost_based, avaiable_index_name[act_mileage_balance_txn_id], estimation info[table_id:1103909674361155, (table_type:1, version:0-1697652010962055-1697652010962055, logical_rc:1010948, physical_rc:1010948), (table_type:7, version:1697652003239449-1697652003239449-1697652034150774, logical_rc:0, physical_rc:0), (table_type:7, version:1697652034150774-1697654499978097-1697656200119339, logical_rc:0, physical_rc:0), (table_type:7, version:1697656200119339-1697656200119339-1697657682768929, logical_rc:0, physical_rc:0), (table_type:5, version:1697656200119339-1697656200119339-1697657682768929, logical_rc:0, physical_rc:0), (table_type:0, version:1697657682768929-1697657682768929-9223372036854775807, logical_rc:0, physical_rc:0)]

D:table_rows:12272897, physical_range_rows:78617471, logical_range_rows:78617471, index_back_rows:0, output_rows:135081, est_method:local_storage, optimization_method=cost_based, avaiable_index_name[act_membership], estimation info[table_id:1103909674339320, (table_type:1, version:0-1697652010962055-1697652010962055, logical_rc:1288811, physical_rc:1288811), (table_type:7, version:1697652001537715-1697652001537715-1697652032336336, logical_rc:0, physical_rc:0), (table_type:7, version:1697652032336336-1697654394078185-1697656198034189, logical_rc:0, physical_rc:0), (table_type:7, version:1697656198034189-1697656198034189-1697657682108509, logical_rc:0, physical_rc:0), (table_type:5, version:1697656198034189-1697656198034189-1697657682108509, logical_rc:0, physical_rc:0), (table_type:0, version:1697657682108509-1697657682108509-9223372036854775807, logical_rc:0, physical_rc:0)]

C:table_rows:131877359, physical_range_rows:796525373, logical_range_rows:796525373, index_back_rows:0, output_rows:796525373, est_method:local_storage, optimization_method=cost_based, avaiable_index_name[act_event], estimation info[table_id:1103909674339265, (table_type:1, version:0-1697652010962055-1697652010962055, logical_rc:13057793, physical_rc:13057793), (table_type:7, version:1697652009045034-1697652009045034-1697652040771265, logical_rc:0, physical_rc:0), (table_type:7, version:1697652040771265-1697654489944006-1697656204189993, logical_rc:0, physical_rc:0), (table_type:7, version:1697656204189993-1697656204189993-1697657684627072, logical_rc:0, physical_rc:0), (table_type:5, version:1697656204189993-1697656204189993-1697657684627072, logical_rc:0, physical_rc:0), (table_type:0, version:1697657684627072-1697657684627072-9223372036854775807, logical_rc:0, physical_rc:0)]

A:table_rows:204989314, physical_range_rows:1293203721, logical_range_rows:1293203721, index_back_rows:0, output_rows:646601860, est_method:local_storage, optimization_method=cost_based, avaiable_index_name[index_Evt_ID,act_transaction], estimation info[table_id:1103909674339272, (table_type:1, version:0-1697652010962055-1697652010962055, logical_rc:21200061, physical_rc:21200061), (table_type:7, version:1697652007746043-1697652007746043-1697652039479452, logical_rc:0, physical_rc:0), (table_type:7, version:1697652039479452-1697654515739071-1697656203322760, logical_rc:0, physical_rc:0), (table_type:7, version:1697656203322760-1697656203322760-1697657684385448, logical_rc:0, physical_rc:0), (table_type:5, version:1697656203322760-1697656203322760-1697657684385448, logical_rc:0, physical_rc:0), (table_type:0, version:1697657684385448-1697657684385448-9223372036854775807, logical_rc:0, physical_rc:0)]

Parameters:
-------------------------------------
{obj:{"DATE":"2999-12-31"}, accuracy:{length:-1, precision:0, scale:0}, flag:0, raw_text_pos:-1, raw_text_len:-1, param_meta:{type:"DATE", collation:"binary", coercibility:"NUMERIC"}}, {obj:{"DECIMAL":"1"}, accuracy:{length:-1, precision:1, scale:0}, flag:0, raw_text_pos:-1, raw_text_len:-1, param_meta:{type:"DECIMAL", collation:"binary", coercibility:"NUMERIC"}}

Note:
-------------------------------------
Degree of Parallelism is 32 because of hint

``` 

### 4.2

```
------ActionDB 4.2 版本
-------ActionDB 4.2 版本
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Query Plan                                                                                                                                                                                          |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| =========================================================================================================                                                                                           |
|  | ID | OPERATOR | NAME | EST.ROWS | EST.TIME(us) |  |
| --- | --- | --- | --- | --- | --- | --- |
|  | 0 | PX COORDINATOR |  | 230 | 558434 |  |
|  | 1 | └─EXCHANGE OUT DISTR | :EX10005 | 230 | 558390 |  |
|  | 2 | └─HASH DISTINCT |  | 230 | 558387 |  |
|  | 3 | └─EXCHANGE IN DISTR |  | 230 | 558386 |  |
|  | 4 | └─EXCHANGE OUT DISTR (HASH) | :EX10004 | 230 | 558384 |  |
|  | 5 | └─MATERIAL |  | 230 | 558381 |  |
|  | 6 | └─HASH DISTINCT |  | 230 | 558381 |  |
|  | 7 | └─HASH SEMI JOIN |  | 230 | 558379 |  |
|  | 8 | ├─EXCHANGE IN DISTR |  | 69323 | 473302 |  |
|  | 9 | │ └─EXCHANGE OUT DISTR (HASH) | :EX10002 | 69323 | 472395 |  |
|  | 10 | │   └─MATERIAL |  | 69323 | 470360 |  |
|  | 11 | │     └─NESTED-LOOP JOIN |  | 69323 | 470360 |  |
|  | 12 | │       ├─EXCHANGE IN DISTR |  | 53831 | 102718 |  |
|  | 13 | │       │ └─EXCHANGE OUT DISTR (BC2HOST) | :EX10001 | 53831 | 101437 |  |
|  | 14 | │       │   └─SHARED HASH JOIN |  | 53831 | 99204 |  |
|  | 15 | │       │     ├─JOIN FILTER CREATE | :RF0000 | 40686 | 98103 |  |
|  | 16 | │       │     │ └─EXCHANGE IN DISTR |  | 40686 | 98103 |  |
|  | 17 | │       │     │   └─EXCHANGE OUT DISTR (BC2HOST) | :EX10000 | 40686 | 97231 |  |
|  | 18 | │       │     │     └─PX BLOCK ITERATOR |  | 40686 | 95712 |  |
|  | 19 | │       │     │       └─TABLE FULL SCAN | D | 40686 | 95712 |  |
|  | 20 | │       │     └─JOIN FILTER USE | :RF0000 | 53828 | 518 |  |
|  | 21 | │       │       └─PX BLOCK ITERATOR |  | 53828 | 518 |  |
|  | 22 | │       │         └─TABLE FULL SCAN | C | 53828 | 518 |  |
|  | 23 | │       └─PX PARTITION ITERATOR |  | 1 | 219 |  |
|  | 24 | │         └─TABLE RANGE SCAN | A(index_Evt_ID) | 1 | 219 |  |
|  | 25 | └─EXCHANGE IN DISTR |  | 3141093 | 75625 |  |
|  | 26 | └─EXCHANGE OUT DISTR (HASH) | :EX10003 | 3141093 | 53093 |  |
|  | 27 | └─PX BLOCK ITERATOR |  | 3141093 | 2687 |  |
|  | 28 | └─TABLE FULL SCAN | T | 3141093 | 2687 |  |
| ========================================================================================================= |  |  |  |  |  |  |
| Outputs & filters: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| 0 - output([INTERNAL_FUNCTION(D.mbr_id(0x7f00cb02b2d0))(0x7f1eb533b900)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 1 - output([INTERNAL_FUNCTION(D.mbr_id(0x7f00cb02b2d0))(0x7f1eb533b900)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| dop=32 |  |  |  |  |  |  |
| 2 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| distinct([D.mbr_id(0x7f00cb02b2d0)]) |  |  |  |  |  |  |
| 3 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 4 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [D.mbr_id(0x7f00cb02b2d0)]), dop=32 |  |  |  |  |  |  |
| 5 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 6 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| distinct([D.mbr_id(0x7f00cb02b2d0)]) |  |  |  |  |  |  |
| 7 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| equal_conds([T.TXN_ID(0x7f00cb042cf0) = A.txn_id(0x7f00cb042fd0)(0x7f00cb0466f0)]), other_conds(nil) |  |  |  |  |  |  |
| 8 - output([D.mbr_id(0x7f00cb02b2d0)], [A.txn_id(0x7f00cb042fd0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 9 - output([D.mbr_id(0x7f00cb02b2d0)], [A.txn_id(0x7f00cb042fd0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [A.txn_id(0x7f00cb042fd0)]), dop=32 |  |  |  |  |  |  |
| 10 - output([D.mbr_id(0x7f00cb02b2d0)], [A.txn_id(0x7f00cb042fd0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 11 - output([D.mbr_id(0x7f00cb02b2d0)], [A.txn_id(0x7f00cb042fd0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| conds(nil), nl_params_([C.evt_id(0x7f00cb025110)(:1)]), use_batch=false |  |  |  |  |  |  |
| 12 - output([D.mbr_id(0x7f00cb02b2d0)], [C.evt_id(0x7f00cb025110)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 13 - output([D.mbr_id(0x7f00cb02b2d0)], [C.evt_id(0x7f00cb025110)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| dop=32 |  |  |  |  |  |  |
| 14 - output([D.mbr_id(0x7f00cb02b2d0)], [C.evt_id(0x7f00cb025110)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| equal_conds([C.mbr_id(0x7f00cb02aff0) = D.mbr_id(0x7f00cb02b2d0)(0x7f00cb02a8f0)]), other_conds(nil) |  |  |  |  |  |  |
| 15 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| RF_TYPE(in, range, bloom), RF_EXPR[D.mbr_id(0x7f00cb02b2d0)] |  |  |  |  |  |  |
| 16 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 17 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| dop=32 |  |  |  |  |  |  |
| 18 - output([D.mbr_id(0x7f00cb02b2d0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 19 - output([D.mbr_id(0x7f00cb02b2d0)]), filter([D.enddt(0x7f00cb029ca0) = cast('29991231', DATE(0, 0))(0x7f00cb028860)(0x7f00cb029560)]), rowset=256 |  |  |  |  |  |  |
| access([D.enddt(0x7f00cb029ca0)], [D.mbr_id(0x7f00cb02b2d0)]), partitions(p[0-348]) |  |  |  |  |  |  |
| is_index_back=false, is_global_index=false, filter_before_indexback[false], |  |  |  |  |  |  |
| range_key([D.__pk_increment(0x7f00cb045400)]), range(MIN ; MAX)always true |  |  |  |  |  |  |
| 20 - output([C.evt_id(0x7f00cb025110)], [C.mbr_id(0x7f00cb02aff0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 21 - output([C.evt_id(0x7f00cb025110)], [C.mbr_id(0x7f00cb02aff0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 22 - output([C.evt_id(0x7f00cb025110)], [C.mbr_id(0x7f00cb02aff0)]), filter([RF_IN_FILTER(C.mbr_id(0x7f00cb02aff0))(0x7f1eb53314b0)], [RF_RANGE_FILTER(C.mbr_id(0x7f00cb02aff0))(0x7f1eb5331ce0)], |  |  |  |  |  |  |
| [RF_BLOOM_FILTER(C.mbr_id(0x7f00cb02aff0))(0x7f1eb5332510)]), rowset=256 |  |  |  |  |  |  |
| access([C.evt_id(0x7f00cb025110)], [C.mbr_id(0x7f00cb02aff0)]), partitions(p[0-348]) |  |  |  |  |  |  |
| is_index_back=false, is_global_index=false, filter_before_indexback[false,false,false], |  |  |  |  |  |  |
| range_key([C.__pk_increment(0x7f00cb045130)]), range(MIN ; MAX)always true |  |  |  |  |  |  |
| 23 - output([A.txn_id(0x7f00cb042fd0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| access all |  |  |  |  |  |  |
| 24 - output([A.txn_id(0x7f00cb042fd0)]), filter([A.txn_typ(0x7f00cb02c4a0) = cast(1, DECIMAL(1, 0))(0x7f00cb02c9f0)(0x7f00cb02bda0)]), rowset=256 |  |  |  |  |  |  |
| access([A.__pk_increment(0x7f00cb044e60)], [A.txn_typ(0x7f00cb02c4a0)], [A.txn_id(0x7f00cb042fd0)]), partitions(p[0-348]) |  |  |  |  |  |  |
| is_index_back=true, is_global_index=false, filter_before_indexback[false], |  |  |  |  |  |  |
| range_key([A.evt_id(0x7f00cb024e30)], [A.__pk_increment(0x7f00cb044e60)]), range(MIN ; MAX), |  |  |  |  |  |  |
| range_cond([A.evt_id(0x7f00cb024e30) = :1(0x7f466d2a9d20)]) |  |  |  |  |  |  |
| 25 - output([T.TXN_ID(0x7f00cb042cf0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 26 - output([T.TXN_ID(0x7f00cb042cf0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [T.TXN_ID(0x7f00cb042cf0)]), dop=32 |  |  |  |  |  |  |
| 27 - output([T.TXN_ID(0x7f00cb042cf0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 28 - output([T.TXN_ID(0x7f00cb042cf0)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| access([T.TXN_ID(0x7f00cb042cf0)]), partitions(p0) |  |  |  |  |  |  |
| is_index_back=false, is_global_index=false, |  |  |  |  |  |  |
| range_key([T.__pk_increment(0x7f00cb044990)]), range(MIN ; MAX)always true |  |  |  |  |  |  |
| Used Hint: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| /*+ |  |  |  |  |  |  |
|  |  |  |  |  |  |  |
| PARALLEL(32) |  |  |  |  |  |  |
| */ |  |  |  |  |  |  |
| Qb name trace: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| stmt_id:0, stmt_type:T_EXPLAIN |  |  |  |  |  |  |
| stmt_id:1, SEL$1 > SEL$6FCAE2AA > SEL$6816FF38 > SEL$4EFC21DE |  |  |  |  |  |  |
| stmt_id:2, SEL$2 |  |  |  |  |  |  |
| Outline Data: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| /*+ |  |  |  |  |  |  |
| BEGIN_OUTLINE_DATA |  |  |  |  |  |  |
| DISTINCT_PUSHDOWN(@"SEL$4EFC21DE") |  |  |  |  |  |  |
| USE_HASH_DISTINCT(@"SEL$4EFC21DE") |  |  |  |  |  |  |
| LEADING(@"SEL$4EFC21DE" ((("dev_edw"."D"@"SEL$1" "dev_edw"."C"@"SEL$1") "dev_edw"."A"@"SEL$1") "dev_tmp"."T"@"SEL$2")) |  |  |  |  |  |  |
| USE_HASH(@"SEL$4EFC21DE" "dev_tmp"."T"@"SEL$2") |  |  |  |  |  |  |
| PQ_DISTRIBUTE(@"SEL$4EFC21DE" "dev_tmp"."T"@"SEL$2" HASH HASH) |  |  |  |  |  |  |
| USE_NL(@"SEL$4EFC21DE" "dev_edw"."A"@"SEL$1") |  |  |  |  |  |  |
| PQ_DISTRIBUTE(@"SEL$4EFC21DE" "dev_edw"."A"@"SEL$1" BC2HOST NONE) |  |  |  |  |  |  |
| USE_HASH(@"SEL$4EFC21DE" "dev_edw"."C"@"SEL$1") |  |  |  |  |  |  |
| PQ_DISTRIBUTE(@"SEL$4EFC21DE" "dev_edw"."C"@"SEL$1" BC2HOST NONE) |  |  |  |  |  |  |
| PX_JOIN_FILTER(@"SEL$4EFC21DE" "dev_edw"."C"@"SEL$1" "dev_edw"."D"@"SEL$1") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "D"@"SEL$1" 32) |  |  |  |  |  |  |
| FULL(@"SEL$4EFC21DE" "D"@"SEL$1") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "C"@"SEL$1" 32) |  |  |  |  |  |  |
| FULL(@"SEL$4EFC21DE" "C"@"SEL$1") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "A"@"SEL$1" 32) |  |  |  |  |  |  |
| INDEX(@"SEL$4EFC21DE" "A"@"SEL$1" "index_Evt_ID") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "T"@"SEL$2" 32) |  |  |  |  |  |  |
| FULL(@"SEL$4EFC21DE" "T"@"SEL$2") |  |  |  |  |  |  |
| UNNEST(@"SEL$2") |  |  |  |  |  |  |
| OUTER_TO_INNER(@"SEL$6FCAE2AA") |  |  |  |  |  |  |
| MERGE(@"SEL$2" > "SEL$6816FF38") |  |  |  |  |  |  |
| PARALLEL(32) |  |  |  |  |  |  |
| OPTIMIZER_FEATURES_ENABLE('4.0.0.0') |  |  |  |  |  |  |
| END_OUTLINE_DATA |  |  |  |  |  |  |
| */ |  |  |  |  |  |  |
| Optimization Info: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| D: |  |  |  |  |  |  |
| table_rows:86619734 |  |  |  |  |  |  |
| physical_range_rows:86619734 |  |  |  |  |  |  |
| logical_range_rows:86619734 |  |  |  |  |  |  |
| index_back_rows:0 |  |  |  |  |  |  |
| output_rows:40685 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[act_membership] |  |  |  |  |  |  |
| stats version:1697452469762002 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| C: |  |  |  |  |  |  |
| table_rows:1170206 |  |  |  |  |  |  |
| physical_range_rows:1170206 |  |  |  |  |  |  |
| logical_range_rows:1170206 |  |  |  |  |  |  |
| index_back_rows:0 |  |  |  |  |  |  |
| output_rows:1170206 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[act_event] |  |  |  |  |  |  |
| stats version:1697452567628639 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| A: |  |  |  |  |  |  |
| table_rows:1947200820 |  |  |  |  |  |  |
| physical_range_rows:1 |  |  |  |  |  |  |
| logical_range_rows:1 |  |  |  |  |  |  |
| index_back_rows:1 |  |  |  |  |  |  |
| output_rows:0 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[index_Evt_ID, act_transaction] |  |  |  |  |  |  |
| unstable_index_name:[act_transaction] |  |  |  |  |  |  |
| stats version:1697452427671572 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| T: |  |  |  |  |  |  |
| table_rows:3141093 |  |  |  |  |  |  |
| physical_range_rows:3141093 |  |  |  |  |  |  |
| logical_range_rows:3141093 |  |  |  |  |  |  |
| index_back_rows:0 |  |  |  |  |  |  |
| output_rows:3141093 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[act_mileage_balance_txn_id] |  |  |  |  |  |  |
| stats version:1697641812140026 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| Plan Type: |  |  |  |  |  |  |
| DISTRIBUTED |  |  |  |  |  |  |
| Note: |  |  |  |  |  |  |
| Degree of Parallelism is 32 because of hint |  |  |  |  |  |  | +-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 182 rows in set (0.02 sec)

``` 

### 4.2 添加hint版本 /*+leading(t,((d,c),a)) full(a)*/ 

```
---------4.2 添加hint版本 /*+leading(t,((d,c),a)) full(a)*/  
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Query Plan                                                                                                                                                                                             |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ====================================================================================================                                                                                                   |
|  | ID | OPERATOR | NAME | EST.ROWS | EST.TIME(us) |  |
| --- | --- | --- | --- | --- | --- | --- |
|  | 0 | PX COORDINATOR |  | 230 | 2710288 |  |
|  | 1 | └─EXCHANGE OUT DISTR | :EX10006 | 230 | 2710244 |  |
|  | 2 | └─HASH DISTINCT |  | 230 | 2710241 |  |
|  | 3 | └─EXCHANGE IN DISTR |  | 230 | 2710239 |  |
|  | 4 | └─EXCHANGE OUT DISTR (HASH) | :EX10005 | 230 | 2710238 |  |
|  | 5 | └─MATERIAL |  | 230 | 2710235 |  |
|  | 6 | └─HASH DISTINCT |  | 230 | 2710235 |  |
|  | 7 | └─HASH RIGHT SEMI JOIN |  | 230 | 2710233 |  |
|  | 8 | ├─JOIN FILTER CREATE | :RF0002 | 3141093 | 75625 |  |
|  | 9 | │ └─EXCHANGE IN DISTR |  | 3141093 | 75625 |  |
|  | 10 | │   └─EXCHANGE OUT DISTR (HASH) | :EX10000 | 3141093 | 53093 |  |
|  | 11 | │     └─PX BLOCK ITERATOR |  | 3141093 | 2687 |  |
|  | 12 | │       └─TABLE FULL SCAN | T | 3141093 | 2687 |  |
|  | 13 | └─EXCHANGE IN DISTR |  | 69323 | 2615194 |  |
|  | 14 | └─EXCHANGE OUT DISTR (HASH) | :EX10004 | 69323 | 2614287 |  |
|  | 15 | └─MATERIAL |  | 69323 | 2612252 |  |
|  | 16 | └─HASH JOIN |  | 69323 | 2612252 |  |
|  | 17 | ├─JOIN FILTER CREATE | :RF0001 | 53831 | 101281 |  |
|  | 18 | │ └─EXCHANGE IN DISTR |  | 53831 | 101281 |  |
|  | 19 | │   └─EXCHANGE OUT DISTR (HASH) | :EX10002 | 53831 | 100640 |  |
|  | 20 | │     └─SHARED HASH JOIN |  | 53831 | 99204 |  |
|  | 21 | │       ├─JOIN FILTER CREATE | :RF0000 | 40686 | 98103 |  |
|  | 22 | │       │ └─EXCHANGE IN DISTR |  | 40686 | 98103 |  |
|  | 23 | │       │   └─EXCHANGE OUT DISTR (BC2HOST) | :EX10001 | 40686 | 97231 |  |
|  | 24 | │       │     └─PX BLOCK ITERATOR |  | 40686 | 95712 |  |
|  | 25 | │       │       └─TABLE FULL SCAN | D | 40686 | 95712 |  |
|  | 26 | │       └─JOIN FILTER USE | :RF0000 | 53828 | 518 |  |
|  | 27 | │         └─PX BLOCK ITERATOR |  | 53828 | 518 |  |
|  | 28 | │           └─TABLE FULL SCAN | C | 53828 | 518 |  |
|  | 29 | └─EXCHANGE IN DISTR |  | 5101 | 2510585 |  |
|  | 30 | └─EXCHANGE OUT DISTR (HASH) | :EX10003 | 5101 | 2510488 |  |
|  | 31 | └─JOIN FILTER USE | :RF0001 | 5101 | 2510270 |  |
|  | 32 | └─JOIN FILTER USE | :RF0002 | 5101 | 2510270 |  |
|  | 33 | └─PX BLOCK ITERATOR |  | 5101 | 2510270 |  |
|  | 34 | └─TABLE FULL SCAN | A | 5101 | 2510270 |  |
| ==================================================================================================== |  |  |  |  |  |  |
| Outputs & filters: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| 0 - output([INTERNAL_FUNCTION(D.mbr_id(0x7f0f9362c230))(0x7f51dc897e10)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 1 - output([INTERNAL_FUNCTION(D.mbr_id(0x7f0f9362c230))(0x7f51dc897e10)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| dop=32 |  |  |  |  |  |  |
| 2 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| distinct([D.mbr_id(0x7f0f9362c230)]) |  |  |  |  |  |  |
| 3 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 4 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [D.mbr_id(0x7f0f9362c230)]), dop=32 |  |  |  |  |  |  |
| 5 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 6 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| distinct([D.mbr_id(0x7f0f9362c230)]) |  |  |  |  |  |  |
| 7 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| equal_conds([T.TXN_ID(0x7f0f93643c50) = A.txn_id(0x7f0f93643f30)(0x7f0f936478e0)]), other_conds(nil) |  |  |  |  |  |  |
| 8 - output([T.TXN_ID(0x7f0f93643c50)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| RF_TYPE(in, range, bloom), RF_EXPR[T.TXN_ID(0x7f0f93643c50)] |  |  |  |  |  |  |
| 9 - output([T.TXN_ID(0x7f0f93643c50)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 10 - output([T.TXN_ID(0x7f0f93643c50)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [T.TXN_ID(0x7f0f93643c50)]), dop=32 |  |  |  |  |  |  |
| 11 - output([T.TXN_ID(0x7f0f93643c50)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 12 - output([T.TXN_ID(0x7f0f93643c50)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| access([T.TXN_ID(0x7f0f93643c50)]), partitions(p0) |  |  |  |  |  |  |
| is_index_back=false, is_global_index=false, |  |  |  |  |  |  |
| range_key([T.__pk_increment(0x7f0f93645b80)]), range(MIN ; MAX)always true |  |  |  |  |  |  |
| 13 - output([D.mbr_id(0x7f0f9362c230)], [A.txn_id(0x7f0f93643f30)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 14 - output([D.mbr_id(0x7f0f9362c230)], [A.txn_id(0x7f0f93643f30)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [A.txn_id(0x7f0f93643f30)]), dop=32 |  |  |  |  |  |  |
| 15 - output([D.mbr_id(0x7f0f9362c230)], [A.txn_id(0x7f0f93643f30)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 16 - output([D.mbr_id(0x7f0f9362c230)], [A.txn_id(0x7f0f93643f30)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| equal_conds([A.evt_id(0x7f0f93625d90) = C.evt_id(0x7f0f93626070)(0x7f0f93625690)]), other_conds(nil) |  |  |  |  |  |  |
| 17 - output([D.mbr_id(0x7f0f9362c230)], [C.evt_id(0x7f0f93626070)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| RF_TYPE(in, range, bloom), RF_EXPR[C.evt_id(0x7f0f93626070)] |  |  |  |  |  |  |
| 18 - output([D.mbr_id(0x7f0f9362c230)], [C.evt_id(0x7f0f93626070)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 19 - output([D.mbr_id(0x7f0f9362c230)], [C.evt_id(0x7f0f93626070)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [C.evt_id(0x7f0f93626070)]), dop=32 |  |  |  |  |  |  |
| 20 - output([D.mbr_id(0x7f0f9362c230)], [C.evt_id(0x7f0f93626070)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| equal_conds([C.mbr_id(0x7f0f9362bf50) = D.mbr_id(0x7f0f9362c230)(0x7f0f9362b850)]), other_conds(nil) |  |  |  |  |  |  |
| 21 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| RF_TYPE(in, range, bloom), RF_EXPR[D.mbr_id(0x7f0f9362c230)] |  |  |  |  |  |  |
| 22 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 23 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| dop=32 |  |  |  |  |  |  |
| 24 - output([D.mbr_id(0x7f0f9362c230)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 25 - output([D.mbr_id(0x7f0f9362c230)]), filter([D.enddt(0x7f0f9362ac00) = cast('29991231', DATE(0, 0))(0x7f0f936297c0)(0x7f0f9362a4c0)]), rowset=256 |  |  |  |  |  |  |
| access([D.enddt(0x7f0f9362ac00)], [D.mbr_id(0x7f0f9362c230)]), partitions(p[0-348]) |  |  |  |  |  |  |
| is_index_back=false, is_global_index=false, filter_before_indexback[false], |  |  |  |  |  |  |
| range_key([D.__pk_increment(0x7f0f936465f0)]), range(MIN ; MAX)always true |  |  |  |  |  |  |
| 26 - output([C.evt_id(0x7f0f93626070)], [C.mbr_id(0x7f0f9362bf50)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 27 - output([C.evt_id(0x7f0f93626070)], [C.mbr_id(0x7f0f9362bf50)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 28 - output([C.evt_id(0x7f0f93626070)], [C.mbr_id(0x7f0f9362bf50)]), filter([RF_IN_FILTER(C.mbr_id(0x7f0f9362bf50))(0x7f51dc8840e0)], [RF_RANGE_FILTER(C.mbr_id(0x7f0f9362bf50))(0x7f51dc884910)], |  |  |  |  |  |  |
| [RF_BLOOM_FILTER(C.mbr_id(0x7f0f9362bf50))(0x7f51dc885140)]), rowset=256 |  |  |  |  |  |  |
| access([C.evt_id(0x7f0f93626070)], [C.mbr_id(0x7f0f9362bf50)]), partitions(p[0-348]) |  |  |  |  |  |  |
| is_index_back=false, is_global_index=false, filter_before_indexback[false,false,false], |  |  |  |  |  |  |
| range_key([C.__pk_increment(0x7f0f93646320)]), range(MIN ; MAX)always true |  |  |  |  |  |  |
| 29 - output([A.txn_id(0x7f0f93643f30)], [A.evt_id(0x7f0f93625d90)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 30 - output([A.txn_id(0x7f0f93643f30)], [A.evt_id(0x7f0f93625d90)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| (#keys=1, [A.evt_id(0x7f0f93625d90)]), dop=32 |  |  |  |  |  |  |
| 31 - output([A.txn_id(0x7f0f93643f30)], [A.evt_id(0x7f0f93625d90)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 32 - output([A.txn_id(0x7f0f93643f30)], [A.evt_id(0x7f0f93625d90)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 33 - output([A.evt_id(0x7f0f93625d90)], [A.txn_id(0x7f0f93643f30)]), filter(nil), rowset=256 |  |  |  |  |  |  |
| 34 - output([A.evt_id(0x7f0f93625d90)], [A.txn_id(0x7f0f93643f30)]), filter([A.txn_typ(0x7f0f9362d400) = cast(1, DECIMAL(1, 0))(0x7f0f9362d950)(0x7f0f9362cd00)], |  |  |  |  |  |  |
| [RF_IN_FILTER(A.evt_id(0x7f0f93625d90))(0x7f51dc888e70)], [RF_RANGE_FILTER(A.evt_id(0x7f0f93625d90))(0x7f51dc8896a0)], [RF_BLOOM_FILTER(A.evt_id(0x7f0f93625d90))(0x7f51dc889ed0)], |  |  |  |  |  |  |
| [RF_IN_FILTER(A.txn_id(0x7f0f93643f30))(0x7f51dc88dc00)], [RF_RANGE_FILTER(A.txn_id(0x7f0f93643f30))(0x7f51dc88e430)], [RF_BLOOM_FILTER(A.txn_id(0x7f0f93643f30))(0x7f51dc88ec60)]), rowset=256 |  |  |  |  |  |  |
| access([A.evt_id(0x7f0f93625d90)], [A.txn_typ(0x7f0f9362d400)], [A.txn_id(0x7f0f93643f30)]), partitions(p[0-348]) |  |  |  |  |  |  |
| is_index_back=false, is_global_index=false, filter_before_indexback[false,false,false,false,false,false,false], |  |  |  |  |  |  |
| range_key([A.__pk_increment(0x7f0f93646050)]), range(MIN ; MAX)always true |  |  |  |  |  |  |
| Used Hint: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| /*+ |  |  |  |  |  |  |
|  |  |  |  |  |  |  |
| LEADING(("t" (("d" "c") "a"))) |  |  |  |  |  |  |
| FULL("a") |  |  |  |  |  |  |
| PARALLEL(32) |  |  |  |  |  |  |
| */ |  |  |  |  |  |  |
| Qb name trace: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| stmt_id:0, stmt_type:T_EXPLAIN |  |  |  |  |  |  |
| stmt_id:1, SEL$1 > SEL$6FCAE2AA > SEL$6816FF38 > SEL$4EFC21DE |  |  |  |  |  |  |
| stmt_id:2, SEL$2 |  |  |  |  |  |  |
| Outline Data: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| /*+ |  |  |  |  |  |  |
| BEGIN_OUTLINE_DATA |  |  |  |  |  |  |
| DISTINCT_PUSHDOWN(@"SEL$4EFC21DE") |  |  |  |  |  |  |
| USE_HASH_DISTINCT(@"SEL$4EFC21DE") |  |  |  |  |  |  |
| LEADING(@"SEL$4EFC21DE" ("dev_tmp"."T"@"SEL$2" (("dev_edw"."D"@"SEL$1" "dev_edw"."C"@"SEL$1") "dev_edw"."A"@"SEL$1"))) |  |  |  |  |  |  |
| USE_HASH(@"SEL$4EFC21DE" ("dev_edw"."D"@"SEL$1" "dev_edw"."C"@"SEL$1" "dev_edw"."A"@"SEL$1")) |  |  |  |  |  |  |
| PQ_DISTRIBUTE(@"SEL$4EFC21DE" ("dev_edw"."D"@"SEL$1" "dev_edw"."C"@"SEL$1" "dev_edw"."A"@"SEL$1") HASH HASH) |  |  |  |  |  |  |
| PX_JOIN_FILTER(@"SEL$4EFC21DE" "dev_edw"."A"@"SEL$1" "dev_tmp"."T"@"SEL$2") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "T"@"SEL$2" 32) |  |  |  |  |  |  |
| FULL(@"SEL$4EFC21DE" "T"@"SEL$2") |  |  |  |  |  |  |
| USE_HASH(@"SEL$4EFC21DE" "dev_edw"."A"@"SEL$1") |  |  |  |  |  |  |
| PQ_DISTRIBUTE(@"SEL$4EFC21DE" "dev_edw"."A"@"SEL$1" HASH HASH) |  |  |  |  |  |  |
| PX_JOIN_FILTER(@"SEL$4EFC21DE" "dev_edw"."A"@"SEL$1" ("dev_edw"."D"@"SEL$1" "dev_edw"."C"@"SEL$1")) |  |  |  |  |  |  |
| USE_HASH(@"SEL$4EFC21DE" "dev_edw"."C"@"SEL$1") |  |  |  |  |  |  |
| PQ_DISTRIBUTE(@"SEL$4EFC21DE" "dev_edw"."C"@"SEL$1" BC2HOST NONE) |  |  |  |  |  |  |
| PX_JOIN_FILTER(@"SEL$4EFC21DE" "dev_edw"."C"@"SEL$1" "dev_edw"."D"@"SEL$1") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "D"@"SEL$1" 32) |  |  |  |  |  |  |
| FULL(@"SEL$4EFC21DE" "D"@"SEL$1") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "C"@"SEL$1" 32) |  |  |  |  |  |  |
| FULL(@"SEL$4EFC21DE" "C"@"SEL$1") |  |  |  |  |  |  |
| PARALLEL(@"SEL$4EFC21DE" "A"@"SEL$1" 32) |  |  |  |  |  |  |
| FULL(@"SEL$4EFC21DE" "A"@"SEL$1") |  |  |  |  |  |  |
| UNNEST(@"SEL$2") |  |  |  |  |  |  |
| OUTER_TO_INNER(@"SEL$6FCAE2AA") |  |  |  |  |  |  |
| MERGE(@"SEL$2" > "SEL$6816FF38") |  |  |  |  |  |  |
| PARALLEL(32) |  |  |  |  |  |  |
| OPTIMIZER_FEATURES_ENABLE('4.0.0.0') |  |  |  |  |  |  |
| END_OUTLINE_DATA |  |  |  |  |  |  |
| */ |  |  |  |  |  |  |
| Optimization Info: |  |  |  |  |  |  |
| ------------------------------------- |  |  |  |  |  |  |
| T: |  |  |  |  |  |  |
| table_rows:3141093 |  |  |  |  |  |  |
| physical_range_rows:3141093 |  |  |  |  |  |  |
| logical_range_rows:3141093 |  |  |  |  |  |  |
| index_back_rows:0 |  |  |  |  |  |  |
| output_rows:3141093 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[act_mileage_balance_txn_id] |  |  |  |  |  |  |
| stats version:1697641812140026 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| D: |  |  |  |  |  |  |
| table_rows:86619734 |  |  |  |  |  |  |
| physical_range_rows:86619734 |  |  |  |  |  |  |
| logical_range_rows:86619734 |  |  |  |  |  |  |
| index_back_rows:0 |  |  |  |  |  |  |
| output_rows:40685 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[act_membership] |  |  |  |  |  |  |
| stats version:1697452469762002 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| C: |  |  |  |  |  |  |
| table_rows:1170206 |  |  |  |  |  |  |
| physical_range_rows:1170206 |  |  |  |  |  |  |
| logical_range_rows:1170206 |  |  |  |  |  |  |
| index_back_rows:0 |  |  |  |  |  |  |
| output_rows:1170206 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[act_event] |  |  |  |  |  |  |
| stats version:1697452567628639 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| A: |  |  |  |  |  |  |
| table_rows:1947200820 |  |  |  |  |  |  |
| physical_range_rows:1947200820 |  |  |  |  |  |  |
| logical_range_rows:1947200820 |  |  |  |  |  |  |
| index_back_rows:0 |  |  |  |  |  |  |
| output_rows:973600410 |  |  |  |  |  |  |
| table_dop:32 |  |  |  |  |  |  |
| dop_method:Global DOP |  |  |  |  |  |  |
| avaiable_index_name:[index_Evt_ID, act_transaction] |  |  |  |  |  |  |
| pruned_index_name:[index_Evt_ID] |  |  |  |  |  |  |
| stats version:1697452427671572 |  |  |  |  |  |  |
| dynamic sampling level:0 |  |  |  |  |  |  |
| Plan Type: |  |  |  |  |  |  |
| DISTRIBUTED |  |  |  |  |  |  |
| Note: |  |  |  |  |  |  |
| Degree of Parallelism is 32 because of hint |  |  |  |  |  |  | +--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 201 rows in set (0.02 sec)
``` 

# 疑惑点: JOIN FILTER

在4.2添加hint后, 算子8, 在访问T的数据时, 使用了JOIN FILTER, 对比一下局部成本: 

```
--- 4.2添加hint
| |5 |        └─MATERIAL                                            |               |230     |558381      |                                                                                           |
| |6 |          └─HASH DISTINCT                                     |               |230     |558381      |                                                                                           |
| |7 |            └─HASH SEMI JOIN                                  |               |230     |558379      |                                                                                           |
| |8 |              ├─EXCHANGE IN DISTR                             |               |69323   |473302      |                                                                                           |
...
| |25|              └─EXCHANGE IN DISTR                             |               |3141093 |75625       |                                                                                           |
| |26|                └─EXCHANGE OUT DISTR (HASH)                   |:EX10003       |3141093 |53093       |                                                                                           |
| |27|                  └─PX BLOCK ITERATOR                         |               |3141093 |2687        |                                                                                           |
| |28|                    └─TABLE FULL SCAN                         |T              |3141093 |2687        |      
 
算子8的参数: 
|   8 - output([T.TXN_ID(0x7f0f93643c50)]), filter(nil), rowset=256                                                                                                                                      |
|       RF_TYPE(in, range, bloom), RF_EXPR[T.TXN_ID(0x7f0f93643c50)]     
 
--- 4.2
| |5 |        └─MATERIAL                                              |        |230     |2710235     |                                                                                                   |
| |6 |          └─HASH DISTINCT                                       |        |230     |2710235     |                                                                                                   |
| |7 |            └─HASH RIGHT SEMI JOIN                              |        |230     |2710233     |                                                                                                   |
| |8 |              ├─JOIN FILTER CREATE                              |:RF0002 |3141093 |75625       |                                                                                                   |
| |9 |              │ └─EXCHANGE IN DISTR                             |        |3141093 |75625       |                                                                                                   |
| |10|              │   └─EXCHANGE OUT DISTR (HASH)                   |:EX10000|3141093 |53093       |                                                                                                   |
| |11|              │     └─PX BLOCK ITERATOR                         |        |3141093 |2687        |                                                                                                   |
| |12|              │       └─TABLE FULL SCAN                         |T       |3141093 |2687        |                                                                                                   |
| |13|              └─EXCHANGE IN DISTR                               |        |69323   |2615194     |     
...
``` 

强制使用hint前后, OB认为访问表T的成本一样, 但:

  - 添加hint后, 在访问表T时, 创建了一个JOIN FILTER (:RF0002), 并在访问表A时使用了这个FILTER: 

```
| |29|                      └─EXCHANGE IN DISTR                       |        |5101    |2510585     |                                                                                                   |
| |30|                        └─EXCHANGE OUT DISTR (HASH)             |:EX10003|5101    |2510488     |                                                                                                   |
| |31|                          └─JOIN FILTER USE                     |:RF0001 |5101    |2510270     |                                                                                                   |
| |32|                            └─JOIN FILTER USE                   |:RF0002 |5101    |2510270     |                                                                                                   |
| |33|                              └─PX BLOCK ITERATOR               |        |5101    |2510270     |                                                                                                   |
| |34|                                └─TABLE FULL SCAN               |A       |5101    |2510270     |      
```
