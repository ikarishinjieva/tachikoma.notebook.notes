---
title: 20240613 - 服务器的docker pull访问本地的翻墙代理
confluence_page_id: 2949670
created_at: 2024-06-13T05:22:03+00:00
updated_at: 2024-06-13T05:22:03+00:00
---

1. 将本地代理clash的配置中, 打开"接受局域网连接"
  2. 将本地代理端口7890, 映射到服务器的7990端口  
  

```
ssh -R 7899:localhost:7890 root@10.186.62.73
``` 
  3. 在服务器上, 查看docker的配置文件, systemctl status docker 显示为: /lib/systemd/system/docker.service
  4. 修改/lib/systemd/system/docker.service, 添加PROXY配置: 

```
Environment="HTTP_PROXY=http://127.0.0.1:7899"
Environment="HTTPS_PROXY=http://127.0.0.1:7899"
Environment="NO_PROXY=localhost,127.0.0.1,::1"
``` 
  5. 刷新配置

```
systemctl daemon-reload
systemctl restart docker
``` 
  6. docker pull可翻墙
