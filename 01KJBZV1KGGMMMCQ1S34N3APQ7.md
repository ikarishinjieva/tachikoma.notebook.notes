---
title: 20250708 - claude code使用
confluence_page_id: 4161592
created_at: 2025-07-08T05:32:19+00:00
updated_at: 2025-07-08T05:32:19+00:00
---

使用 "野卡" 的虚拟信用卡付费

安装claude code后, 将执行文件做成脚本, 以套上proxy: 

```
#!/bin/bash
ALL_PROXY=http://127.0.0.1:7890 /usr/lib/node_modules/@anthropic-ai/claude-code/cli.js
```
