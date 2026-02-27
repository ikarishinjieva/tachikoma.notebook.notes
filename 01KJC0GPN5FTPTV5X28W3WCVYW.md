---
title: 20260220 - 做一个个人confluence的替代品
confluence_page_id: 4620498
created_at: 2026-02-19T17:04:23+00:00
updated_at: 2026-02-21T17:44:21+00:00
---

先跟AI讨论需求: <https://poe.com/s/6H2mSisrNarcP0pAzPRk>

下一步: 

  - 尝试使用 superpowers
  - 尝试使用一个浏览器skill来进行测试, skyvern? 
  - 尝试 Agent teams ? 

### 尝试使用 superpowers

在cursor的marketplace中, 可以直接在项目中添加 superpowers

### 浏览器测试, 仍然使用playwright MCP

遇到很多问题, 重新安装步骤: 

```
npx playwright install chrome
 
MCP配置: 
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--headless", "--browser", "chrome", "--no-sandbox"]
    }
  }
}
``` 

配置后, 得重启 cursor (不确定是否因为cursor旧版本有bug), 才能正确使用浏览器
