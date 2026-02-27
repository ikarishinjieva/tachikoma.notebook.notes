---
title: 20240625 - 使用Clash UI
confluence_page_id: 2949823
created_at: 2024-06-25T07:05:54+00:00
updated_at: 2024-06-25T07:31:32+00:00
---

启动一个没有安全措施的Chrome: 

```
open -n -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --args --user-data-dir="/tmp/chrome_dev_test" --disable-web-security
``` 

访问: <http://clash.razord.top/#/proxies>, 可使用GUI连接本地的clash

测试proxy: 

```
wget -e use_proxy=yes -e http_proxy=http://127.0.0.1:7890  www.google.com
```
