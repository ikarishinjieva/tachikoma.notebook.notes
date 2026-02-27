---
title: 20260220 - 做一个个人confluence的替代品
created_at: 2026-02-19T17:04:23+00:00
updated_at: 2026-02-27T07:49:07.866880+00:00
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

### 开发过程

完全由AI自主开发, 人类只负责许愿和验收, 基础验收由playwright MCP处理

一些和confluence的对齐操作由人类提出

superpowers 在其中完全没有作用, 可能是因为需求简单, 或者 sonnet 4.6 已经自己具有了同样技能, 不调用额外skills

碰到一次, bug修复 由 sonnet 和 opus 都处理很坎坷, 由gpt 5 codex 一次处理成功

### git仓库

项目: <https://github.com/ikarishinjieva/tachikoma.notebook>

数据: <https://github.com/ikarishinjieva/tachikoma.notebook.notes>

### 编译环境

放在 ssh -p 24922 root@183.196.130.56 的 /data/huangyan/tachikoma.notebook/

### 生产环境

部署在阿里云confluence服务器上

通过cursor部署, 部署参考文档在 requirement_and_design 目录中