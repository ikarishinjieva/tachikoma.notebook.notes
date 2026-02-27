---
title: 20251002 - 用移动硬盘安装第二个操作系统ubuntu
confluence_page_id: 4358698
created_at: 2025-10-02T11:01:14+00:00
updated_at: 2025-10-02T11:01:14+00:00
---

好的！下面给你一套从准备到安装、使用与恢复的完整步骤，确保在不改动内置硬盘的前提下，把 Ubuntu 安装到外置 SSD/移动硬盘上，并且引导器也写到外置盘。这样拔掉外置盘就能回到原来的 Windows 环境。

一. 目标与原则

  - 目标：Ubuntu 系统完整安装在外置 SSD（含 EFI/引导、系统、交换/数据分区）。
  - 原则：
    - 不写入内置硬盘的分区和引导。
    - BIOS/UEFI 启动顺序不改或可恢复。
    - 插外置盘时可从外置盘启动 Ubuntu；拔掉时自动回到原 Windows。

二. 准备工作

  1. 备份数据（强烈建议）

  - Windows 重要资料备份到云或另一块盘。
  - 可制作 Windows 恢复盘（控制面板 > 恢复 > 创建恢复驱动器）。

  1. 确认固件与磁盘信息

  - 大多数现代笔记本是 UEFI+GPT。按住 Shift 重启 > 疑难解答 > 高级选项 > UEFI 固件设置，可以进入 BIOS 查看。
  - 记录内置硬盘型号容量，避免安装时选错磁盘。

  1. 准备外置存储

  - 建议：NVMe SSD + USB 3.1/3.2/USB‑C 固态硬盘盒。
  - 容量：至少 64 GB，推荐 ≥256 GB。
  - 性能越好，系统体验越接近内置盘。

  1. 下载镜像与制作启动盘

  - Ubuntu LTS（22.04/24.04 均可），从官网获取 ISO。
  - 用 Rufus（Windows）制作安装U盘：
    - 设备：你的U盘（≥8 GB）
    - 启动类型：选择 Ubuntu ISO
    - 分区类型：GPT
    - 目标系统：UEFI（非 CSM）
    - 文件系统：FAT32
    - 开始写入，完成后弹出。

  1. BIOS/UEFI 设置（如有）

  - 关闭快速启动（Windows 电源选项里也关“启用快速启动”）。
  - 若 Secure Boot 导致引导问题，可暂时关闭；Ubuntu 新版通常兼容 Secure Boot。
  - 启用从 USB 启动。记住启动菜单热键（常见 F12/F11/ESC）。

三. 安装前的风险控制

  - 拔掉除“外置 SSD”和“安装U盘”之外的其他外设/存储，避免误选。
  - 暂停 BitLocker（如系统盘已加密），以免后续重启验证麻烦：控制面板 > BitLocker 驱动器加密 > 暂停保护。

四. 开始安装 Ubuntu 到外置 SSD

  1. 从安装U盘启动

  - 插入安装U盘和外置 SSD。
  - 开机按启动菜单键，选择“UEFI: 你的U盘名称”。
  - 选择“Try or Install Ubuntu”。

  1. 进入安装向导

  - 语言/键盘：简体中文/适用布局。
  - 网络：可先连 Wi‑Fi，便于安装时下载更新与驱动。
  - 安装类型：选择“正常安装”，勾选“安装第三方软件”（有独显时建议勾选）。
  - 重要：当出现“安装类型”（分区）页面时，选择“其他选项”（手动分区）。不要用“与 Windows 共存/擦除磁盘”等自动项，防止写到内置盘。

  1. 精确识别磁盘

  - 在分区器界面，磁盘以 /dev/sdX 或 /dev/nvmeXn1 显示：
    - 内置 NVMe 常为 /dev/nvme0n1（容量显示与你笔记本内置盘相符）。
    - 外置 SSD 常为 /dev/sda 或 /dev/nvme1n1（看容量与品牌判断）。
  - 确认后，所有操作只在外置盘进行。

  1. 在外置 SSD 上创建分区（GPT 建议）

  - 若外置盘已有分区，选中每个分区点击“删除”，整理为“空闲空间”。
  - 创建分区建议：
    - EFI System Partition (ESP)
      - 大小：300–512 MB
      - 类型：EFI System
      - 用途：挂载点 /boot/efi
      - 文件系统：FAT32
    - 根分区 /
      - 大小：50–100+ GB（视使用而定）
      - 文件系统：ext4
      - 挂载点：/
    - 交换空间（两种选一）
      - 方案A：单独 swap 分区，大小 4–8 GB（若需休眠，至少与内存相当）
      - 方案B：不建分区，后续在系统内用 swapfile，更灵活（推荐）
    - 可选数据分区
      - /home 单独分区（如长期使用更方便备份/重装）
  - 关键设置：安装引导装载器的设备
    - 底部“将引导装载器安装到：”务必选择外置 SSD 本体而非某个分区，如：
      - 外置 SATA/USB SSD：/dev/sda
      - 外置 NVMe：/dev/nvme1n1
    - 切勿选到内置盘（/dev/nvme0n1 等）。

  1. 开始安装

  - 检查分区表与引导目标确认无误后继续。
  - 选择时区、创建用户名密码，等待安装完成。
  - 完成后“立即重启”，按提示拔出安装U盘，但保留外置 SSD 连接。

五. 首次启动与验证

  1. 从外置盘引导

  - 重启时进入启动菜单，选择“UEFI: 外置SSD（或其 Boot 选项）”。
  - 若 BIOS 启动项中出现“ubuntu”，将其与外置盘绑定（部分机器会自动识别为外置设备的 EFI 条目）。

  1. 进入 Ubuntu 后的初始化

  - 更新系统：
    - sudo apt update && sudo apt full-upgrade -y
  - 安装常用工具：
    - sudo apt install -y build-essential git curl vim htop net-tools
  - 若未创建 swap 分区且需要交换文件：
    - sudo fallocate -l 4G /swapfile
    - sudo chmod 600 /swapfile
    - sudo mkswap /swapfile
    - echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
    - sudo swapon /swapfile
  - 可安装显卡驱动（NVIDIA）：
    - sudo ubuntu-drivers autoinstall
    - 装完重启。

  1. 验证不影响内置 Windows

  - 关机，拔掉外置 SSD，再开机应直接进入 Windows。
  - 插回外置 SSD，进启动菜单选择外置盘，即进入 Ubuntu。

六. 常见问题与排错

  - 开机仍直接进 Windows，找不到外置 Ubuntu：
    - 用启动菜单选择外置盘的 UEFI 启动项。
    - 在 BIOS 中确保 USB 启动开启；如有“EFI USB Device/ubuntu（外置）”条目，置前。
  - 安装时没看到“将引导装载器安装到”选项：
    - 在“其他选项”手动分区界面底部才有该选项；若使用自动安装将可能写入内置盘，请返回重选。
  - Secure Boot 阻止第三方驱动：
    - 可暂时关闭 Secure Boot；或使用已签名驱动（Ubuntu 新版通常支持）。
  - Windows 写入外置盘的“休眠/快速启动”干扰：
    - 确保 Windows 关闭“快速启动”；如使用外置盘与 Windows 共享分区时，避免 NTFS 在休眠状态下挂载。

七. 使用与性能建议

  - 接口优先级：USB‑C/Thunderbolt > USB 3.2 > USB 3.0。
  - 文件系统：ext4 稳定且足够，若大量小文件与断电安全需求高，可考虑 btrfs（配合快照工具）。
  - 电源管理：笔记本在电池模式下，外置盘可能掉速；建议连接电源适配器。
  - 备份：用 Timeshift（ext4）或 btrfs 快照定期备份到同盘或另一盘。

八. 恢复原样/卸载

  - 最简单：不插外置 SSD，电脑即回到原 Windows，内置盘未改动。
  - 若 BIOS 中新增了“ubuntu”引导项但你想清理：
    - 进 BIOS 删除该 Boot Entry，或在 Windows 下使用 bcdedit/UEFI 管理工具清理。
  - 外置盘擦除：
    - 用 Windows“磁盘管理”或 Diskpart 清除分区，或在 Ubuntu 下 sudo wipefs -a /dev/sdX 后重新分区。

九. 可选高级配置

  - 全盘加密（LUKS）：安装时选择“加密 Ubuntu 安装”，同样确保引导器与 ESP 在外置盘；安全性更高，但开机需输入密钥。
  - 多系统多引导：可在同一外置盘上安装多个 Linux 发行版，每个系统共享一个 ESP，但仍全部在外置盘内。

如果你提供笔记本型号、是否有独显（NVIDIA/AMD/Intel）、以及外置盘型号与容量，我可以为你量身给出分区大小建议、驱动注意事项与 BIOS 选项名称对应。
