---
title: 20210628 - 公司上网
confluence_page_id: 1147055
created_at: 2021-06-28T05:20:38+00:00
updated_at: 2023-09-15T03:20:28+00:00
---

ALL_PROXY=[https://mgm-ntidc.actionsky.com:40000/y2Wt6ji8xGKFLv39_pac](<https://mgm.ntidc.actionsky.com:40000/y2Wt6ji8xGKFLv39_pac> "https://mgm.ntidc.actionsky.com:40000/y2Wt6ji8xGKFLv39_pac")

ALL_PROXY=socks5://[mgm-ntidc.actionsky.com:10800](<http://mgm.ntidc.actionsky.com:10800/> "http://mgm.ntidc.actionsky.com:10800") docker pull …

让服务器使用笔记本上的代理: 

ssh -R 7890:localhost:7890 [root@10.186.16.136](<mailto:root@10.186.16.136>)

(在笔记本上执行, 将服务器上的7890端口转到本地代理)

pip代理: 

```
pip install pysocks -i https://pypi.tuna.tsinghua.edu.cn/simple
```
