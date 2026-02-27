---
note: 01KJBZEMNJW3ZAV821WQ5N70HB.md
title: 20240822 - 试用 lavague
indexed_at: 2026-02-27T11:13:32.987141+00:00
---

用户要求我分析一篇关于试用 LaVague 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20240822 - 试用 lavague
2. 内容主要是：
   - 在服务器 10.186.62.73 上安装和测试 LaVague（一个 Web 自动化框架）
   - 使用 conda 创建环境
   - 安装 lavague 包
   - 运行测试代码使用 SeleniumDriver
   - 遇到 Chrome 启动失败的错误
   - 错误原因是缺少 libgbm.so.1 库
   - 解决方案是安装 libgbm1

3. 关联笔记分析：
   - 这篇笔记是关于 LaVague（Web 自动化测试框架）的试用记录
   - 查看其他笔记列表，大部分是关于 MySQL、Clickhouse、数据库相关的笔记
   - 没有明显与 Web 自动化、Selenium、LaVague 相关的笔记
   - 所以关联笔记应该是「无」

让我生成索引：

## 标签
LaVague, Web 自动化, Selenium, Chrome, 环境配置, 故障排查

## 摘要
记录在 Linux 服务器上安装和试用 LaVague Web 自动化框架的过程。遇到 Chrome 无法启动的问题，原因是缺少 libgbm.so.1 共享库，通过安装 libgbm1 解决。
## 标签
LaVague, Web 自动化, Selenium, Chrome, 环境配置, 故障排查

## 摘要
记录在 Linux 服务器上安装和试用 LaVague Web 自动化框架的过程。遇到 Chrome 无法启动的问题，原因是缺少 libgbm.so.1 共享库，通过安装 libgbm1 解决。

## 关键概念
- LaVague: 基于大模型的 Web 自动化测试框架
- SeleniumDriver: LaVague 使用的 Selenium 浏览器驱动
- WorldModel: LaVague 中负责理解网页状态的核心组件
- ActionEngine: LaVague 中负责执行操作的核心组件
- libgbm: Linux 下图形缓冲管理库，Chrome 无头模式依赖

## 关联笔记
无
