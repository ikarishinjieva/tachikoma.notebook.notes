---
title: 20260102 - 对玄雅机械臂进行烧录
confluence_page_id: 4359503
created_at: 2026-01-02T04:59:47+00:00
updated_at: 2026-01-04T04:55:35+00:00
---

# 问题

机械臂拆除了末端执行器, 更换为机械手, 机械臂自检不工作 (末端没有连接到有效舵机上)

与售后沟通, 需要烧录新的系统

资料: 

  - [St-link bin烧录方式 (1).pdf](/assets/01KJC09JK5QH3MKD397FPJGF32/St-link%20bin%E7%83%A7%E5%BD%95%E6%96%B9%E5%BC%8F%20%281%29.pdf)
  - [aliciDuo(1).bin](/assets/01KJC09JK5QH3MKD397FPJGF32/aliciDuo%281%29.bin)

淘宝购买: Stlink/v2 + (1.0mm 4P转杜邦2.54-1P 线)

  - 机械臂一端调试端口为 SH-1.0-4P, Stlink一端为杜邦口. 为了对线序, 所以用杜邦1P口, 这样容易调整

按照文档中的线序说明, 将端口连接好.

在windows中, 会报错mfc140.dll 找不到. 用360安装 visual c++全家桶, 可正常运行. 

在界面上, 将地址调整为 0x08020000, 进行Program verify, 弹出窗口中再次将地址调整为 0x08020000

烧录后, 可以在脱离末端执行器的情况下, 进行初始化自检
