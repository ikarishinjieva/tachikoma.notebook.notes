---
note: 01KJBZTE7KCAE5K5HXHQ091CBR.md
title: 20250615 - 重装旧的MAC osx (2013 Early)
indexed_at: 2026-02-27T11:40:27.035601+00:00
---

用户要求我分析一篇关于重装 Mac OS X 的笔记，并生成索引元数据。让我仔细分析笔记内容：

笔记标题：20250615 - 重装旧的 MAC osx (2013 Early)

笔记内容核心：
1. 目标：安装 Cursor 需要 Mac OS X 11，但笔记本无法直接升级
2. 通过恢复模式重装到 10.8 Tiger
3. 使用 OpenCore-Legacy-Patcher 创建安装移动硬盘
4. Big Sur 可以安装，Monterey 失败
5. HomeBrew 不再支持 Big Sur，改用 MacPorts

标签应该包括：MacOS、系统重装、OpenCore-Legacy-Patcher、Big Sur、MacBook Pro、兼容性

摘要：记录在 MacBook Pro 2013 Early 机型上通过 OpenCore-Legacy-Patcher 工具重装 macOS 的过程。成功安装 Big Sur，但 Monterey 安装失败，最终使用 MacPorts 替代 HomeBrew。

关键概念：
- OpenCore-Legacy-Patcher: 用于在旧 Mac 硬件上安装新版 macOS 的补丁工具
- Big Sur: macOS 11 版本，可在此机型上安装
- MacPorts: 包管理工具，替代不再支持 Big Sur 的 HomeBrew
- 恢复模式: 通过 Command+R 进入的 macOS 恢复环境

关联笔记：查看其他笔记列表，主要是关于 MySQL、Clickhouse、Java、系统性能等技术内容，与这篇 MacOS 重装的笔记没有明显关联。

所以关联笔记应该是「无」。## 标签
MacOS, 系统重装, OpenCore-Legacy-Patcher, Big Sur, MacBook Pro, 包管理工具

## 摘要
记录在 MacBook Pro 2013 Early 机型上通过 OpenCore-Legacy-Patcher 工具重装 macOS 的过程，目标是安装支持 Cursor 的 macOS 11。成功安装 Big Sur 但 Monterey 失败，因 HomeBrew 不再支持 Big Sur 而改用 MacPorts。

## 关键概念
- OpenCore-Legacy-Patcher: 用于在旧 Mac 硬件上安装新版 macOS 的引导补丁工具
- Big Sur: macOS 11 版本，可在此机型上通过补丁工具安装
- MacPorts: 包管理工具，作为 HomeBrew 的替代方案
- 恢复模式: 通过 Command+R 进入的 macOS 系统恢复环境

## 关联笔记
无
