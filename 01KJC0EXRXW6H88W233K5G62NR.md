---
title: 20260212 - 测试intel realsense D405
confluence_page_id: 4620479
created_at: 2026-02-12T03:25:17+00:00
updated_at: 2026-02-12T03:25:17+00:00
---

# 安装

按照 <https://github.com/realsenseai/librealsense/blob/master/doc/distribution_linux.md> 的说明, 安装SDK

在桌面上, 打开console, 启动 realsense-viewer

# 测试-USB 2.1

usb 2.1, 分辨率 848 * 480

![image2026-2-12 11:1:30.png](/assets/01KJC0EXRXW6H88W233K5G62NR/image2026-2-12%2011%3A1%3A30.png)

近距离 15cm: 不能测距的点少

![image2026-2-12 11:1:45.png](/assets/01KJC0EXRXW6H88W233K5G62NR/image2026-2-12%2011%3A1%3A45.png)

近距离 10cm: 不能测距的点变多

![image2026-2-12 11:4:27.png](/assets/01KJC0EXRXW6H88W233K5G62NR/image2026-2-12%2011%3A4%3A27.png)

usb 2.1, 分辨率 1280 * 720, 近距离 10cm, 效果差不多: 

![image2026-2-12 11:11:53.png](/assets/01KJC0EXRXW6H88W233K5G62NR/image2026-2-12%2011%3A11%3A53.png)

usb 3.0: 只是FPS增加, 效果没有进一步提升: 

![image2026-2-12 11:25:7.png](/assets/01KJC0EXRXW6H88W233K5G62NR/image2026-2-12%2011%3A25%3A7.png)
