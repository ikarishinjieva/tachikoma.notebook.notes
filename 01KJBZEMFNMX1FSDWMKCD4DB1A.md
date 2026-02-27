---
title: 20240824 - 测试使用 playwright
confluence_page_id: 3146246
created_at: 2024-08-24T04:13:28+00:00
updated_at: 2024-08-24T04:24:52+00:00
---

环境: 

10.186.58.23

初始化环境: 

```
conda create --name ai-tester
``` 

python版本: 3.12.4

简单代码: 

```
from playwright.sync_api import sync_playwright

width = 1080
height = 1080
headless = False

p = sync_playwright().__enter__()

user_agent = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"

args = [
    "--disable-web-security",
    "--disable-site-isolation-trials",
    "--disable-notifications",
]

browser = p.chromium.launch(
    headless=headless,
    args=args,
)
context = browser.new_context(user_agent=user_agent)

page = context.new_page()
page.set_viewport_size({"width": width, "height": height})

page.goto("http://www.baidu.com")

import time
time.sleep(60)

``` 

需要安装浏览器: 

```
playwright install-deps
playwright install
``` 

  
在Mac上启动Xquatz

ssh -XY -l root 10.186.58.23 启用转发

执行脚本, 可正常运行
