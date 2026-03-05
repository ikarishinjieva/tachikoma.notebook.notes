---
note: 01KJBZV2RVXXM4F6GRNR83GD9G.md
title: 20250709 - 弘讯的数据分析
indexed_at: 2026-03-05T11:45:45.544230+00:00
---

## 标签
弘讯科技, 数据分析, 数据库结构, devicedatanew, devicedatametas, 注塑机

## 摘要
分析弘讯科技 TPC2.0 数据库结构，重点研究 devicedatanew（实时数据）和 devicedatametas（元数据）表的关联关系。通过 DataID 和 machine_id 追踪设备监控参数的含义及机器信息。

## 关键概念
- devicedatanew: 存储设备实时监控数据，包含 id、machine_id、DataID、value、updatetime 等字段
- devicedatametas: 设备数据元数据表，定义 DataID 对应的参数名称和描述（如"托模退 1 段位置"）
- DataID: 数据标识符，用于关联实时数据与元数据表
- machine_id: 机器设备标识，用于关联 machine 和 machine_extra 表获取设备信息


- 关联笔记
- 01KJBZV5A3J7TE29SPA3J0HRJ3.md: 同一系列的后续分析，整理生产过程相关数据的时间线
- 01KJBZV3GZZ6MQD2PZXET9GAKM.md: 记录弘讯数据分析中发现的问题## 标签
弘讯科技，数据分析，数据库结构，devicedatanew，devicedatametas，注塑机

## 摘要
分析弘讯科技 TPC2.0 数据库结构，重点研究 devicedatanew（实时数据）和 devicedatametas（元数据）表的关联关系。通过 DataID 和 machine_id 追踪设备监控参数的含义及机器信息。

## 关键概念
- devicedatanew: 存储设备实时监控数据，包含 id、machine_id、DataID、value、updatetime 等字段
- devicedatametas: 设备数据元数据表，定义 DataID 对应的参数名称和描述（如"托模退 1 段位置"）
- DataID: 数据标识符，用于关联实时数据与元数据表
- machine_id: 机器设备标识，用于关联 machine 和 machine_extra 表获取设备信息

## 关联笔记
- 01KJBZV5A3J7TE29SPA3J0HRJ3.md: 同一系列的后续分析，整理生产过程相关数据的时间线
- 01KJBZV3GZZ6MQD2PZXET9GAKM.md: 记录弘讯数据分析中发现的问题
