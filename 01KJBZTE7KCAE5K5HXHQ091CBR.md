---
title: 20250615 - 重装旧的MAC osx (2013 Early)
confluence_page_id: 4161542
created_at: 2025-06-14T17:05:42+00:00
updated_at: 2025-06-26T05:14:47+00:00
---

目标: 要安装Cursor, 最低需要Mac OSx 11.

但笔记本升级不到最新系统. 通过App store升级到可行的最高版本, 升级失败导致机器不可用

开机后按住command+R, 进入恢复模式, 清理所有磁盘, 然后重装Mac OSX 到10.8 Tiger版本

根据 <https://support.apple.com/en-us/102662>, 下载Yosemite 10.10的安装包尝试升级 (不必要)

使用另外的方法: 在另外一台笔记本上, 使用OpenCore-Legacy-Patcher, 只做一个Mac OSX的安装移动硬盘

使用这个安装移动硬盘, 按照OpenCore-Legacy-Patcher的说明, 进行安装

  - Big Sur可以在笔记本 (Macbook Pro 2013 early)进行安装
  - Monterey会安装失败

HomeBrew不再支持BigSur, 转而使用MacPorts
