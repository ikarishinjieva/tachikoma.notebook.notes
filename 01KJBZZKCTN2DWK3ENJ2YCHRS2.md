---
title: 20250908 - cursor使用海外大模型
confluence_page_id: 4358458
created_at: 2025-09-08T05:00:48+00:00
updated_at: 2025-10-10T03:24:44+00:00
---

1. 将http协议改成1.1
     1. ![image2025-10-10 11:24:18.png](/assets/01KJBZZKCTN2DWK3ENJ2YCHRS2/image2025-10-10%2011%3A24%3A18.png)
  2. 在IDE的配置文件中, 配置如下字段: 

```
  "http.proxy": "socks5://127.0.0.1:7890",
  "http.proxyStrictSSL": false,
  "http.proxySupport": "on",
```
