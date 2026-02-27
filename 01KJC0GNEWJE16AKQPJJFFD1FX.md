---
title: 20260214 - opencode的使用
confluence_page_id: 4620493
created_at: 2026-02-19T06:41:47+00:00
updated_at: 2026-02-19T06:45:07+00:00
---

API代理使用: <https://yansd666.com/>

使用服务器: 

ssh -p 24922 [root@183.196.130.56](<mailto:root@183.196.130.56>)

5cJmSdH^Vp

安装: 

```
export https_proxy=http://127.0.0.1:7990 http_proxy=http://127.0.0.1:7990 all_proxy=socks5://127.0.0.1:7990
curl -fsSL https://opencode.ai/install | bash
``` 

设置配置文件: 

```
新建 ~/.config/opencode/opencode.json : 
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "anthropic": {
      "options": {
        "baseURL": "https://yansd666.com/v1"
      }
    }
  }
}

新建 ~/.local/share/opencode/auth.json :

{
  "anthropic": {
    "type": "api",
    "key": "sk-4F4yg7quytAMfrKscrdMUiv5kL1Qj1HmcbmukSwKnJPqNs2I"
  }
}
``` 

在opencode中, 使用 /models 切换模型

项目代码仓库: [git@github.com](<mailto:git@github.com>):ikarishinjieva/[mat_path.ai](<http://mat_path.ai/>).git

导出session: /export

继续对话: 

```
opencode session list
opencode -s xxxx
```
